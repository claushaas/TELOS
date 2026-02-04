# Pendências e Decisões Abertas — TELOS v0.1

> [SOURCE] Este arquivo existe para registrar o que a SPEC v0.1 (ver [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md)) **não fixa** (ainda), evitando invenção e drift.

## Legenda de Status

- **[RESOLVED]** — Decisão tomada, aguardando implementação/documentação
- **[PENDING]** — Ainda necessita decisão ou confirmação
- **[DECISION]** — Registro da solução escolhida para itens resolvidos

---

## P0 — Bloqueadores (segurança + auditabilidade)

### [RESOLVED] Protocolo de intenção explícita + confirmação (Lince executor)

- **Priority:** P0
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** execução determinística via Lince (CRUD), especialmente ações irreversíveis/alto impacto.

#### [DECISION] Solução escolhida

- **Canal autorizado:** somente **Lince chat** e **Clarify session** (com `correlationId`). Tudo fora disso é read-only ou vira RAW.
- **Confirmação simples:** mutações de 1 entidade com reversão clara (ex.: completar 1 Energeia).
- **Confirmação dupla:**
  - Deleção (mesmo tombstone)
  - Mudança em massa (N>1)
  - Promoção RAW→canônico que cria/relaciona várias entidades
  - Qualquer ação com ambiguidade na resolução do alvo

- **Definition of Done (DoD):**
  - [ ] Contrato documentado em `contratos.md`.
  - [ ] Fluxo documentado em `fluxos.md`.
  - [ ] Implementação registra auditoria antes/depois e recusa executar fora das regras.

---

### [RESOLVED] Resolução de referência (alvos humanos / "tarefa X")

- **Priority:** P0
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** comandos do SER que referenciam entidades por nome/trecho/path (ex.: completar Energeia, criar Poiesis dentro de Ergon).

#### [DECISION] Solução escolhida (Pipeline de 3 estágios)

1. **ID / path / canonical_key** → executa direto (logando).
2. **Busca determinística** (slug/title exato dentro do escopo recente) → se 1 resultado, executa; senão confirma.
3. **Busca semântica** (Vector) → sempre retorna **candidatos** (top 3) e pede confirmação se não houver match único forte.

- **"Match forte":** 1 candidato com score acima do limiar **e** sem empates próximos; se não, confirmação obrigatória.

- **Definition of Done (DoD):**
  - [ ] Contrato documentado em `contratos.md` (incluindo definição de `human_ref`).
  - [ ] Fluxo "resolução de alvo + confirmação" em `fluxos.md`.
  - [ ] Auditoria inclui `human_ref`/candidatos/decisão (quando aplicável).

---

### [RESOLVED] Conflito: checkbox do Obsidian ↔ status canônico (Energeia)

- **Priority:** P0
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** sincronização Vault↔Postgres e consistência de status de Energeia.

#### [DECISION] Solução escolhida

- **Checkbox nunca é canal autorizado no beta.**
- Se checkbox mudar: gerar evento "vault-change-detected" + criar/atualizar um **RawItem** do tipo "sinal de conflito" para Clarify.
- **Exceção futura (não beta):** só se a alteração vier de uma automação assinada pelo sistema (comentário/marker com origem).

- **Definition of Done (DoD):**
  - [ ] Contrato documentado em `contratos.md`.
  - [ ] Fluxo de conflito documentado em `fluxos.md`.
  - [ ] Auditoria registra conflitos e resolução.

---

### [RESOLVED] Auditoria mínima + reversibilidade (AuditEvent, `auditRef`, undo/tombstone)

- **Priority:** P0
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** qualquer mutação canônica (promoção RAW, CRUD, sync, indexação).

#### [DECISION] Solução escolhida

- **Campos obrigatórios no AuditEvent:** `id, createdAt, actor, source, action, entityType, entityId, after, correlationId`
- **Campos recomendados:** `before` (quando update/delete), `reason`, `rawItemId`, `path`
- **Reversibilidade:**
  - **Sempre tombstone** para Poiesis/Energeia (`deletedAt`) no beta.
  - **Undo:** "undo sempre enquanto tombstoned" (restaurar = limpar `deletedAt`) — simples e auditável.

- **Definition of Done (DoD):**
  - [ ] Modelo conceitual em `modelo-de-dados.md` (AuditEvent e campos).
  - [ ] Contrato de audit trail em `contratos.md`.
  - [ ] Implementação registra eventos com `correlationId` e suporta tombstone/undo conforme regra definida.

---

### [RESOLVED] Promoção RAW → Canônico com rastreabilidade

- **Priority:** P0
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** sessões Clarify e qualquer promoção/anexo/decomposição que altere ontologia.

#### [DECISION] Solução escolhida

- **Tabela `RawPromotion`** (não polui entidades e preserva múltiplas promoções/decomposições).
- **Regra de decomposição:**
  - 1 `RawItem` pode gerar N registros em `RawPromotion`.
  - `RawItem.statusInternal` vira `placed` + `placedEntityType/Id` **apenas** se for 1:1; se 1:N, deixar `placed*` vazio e depender de `RawPromotion`.

- **Definition of Done (DoD):**
  - [ ] Contrato em `contratos.md`.
  - [ ] Modelo em `modelo-de-dados.md` (tabela `RawPromotion`).
  - [ ] Auditoria amarra RawItem ↔ entidade(s) canônica(s).

---

## P1 — Infra do plano de memória (Vault / espelho / chunking)

### [RESOLVED] Detecção de mudanças no Vault + gatilho canônico

- **Priority:** P1
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** espelho do Vault, indexação e recuperação contextual confiável.

#### [DECISION] Solução escolhida

- **Watcher local** (fs events) + fallback **polling por hash** (sha256 do arquivo) em intervalos.
- **"Mudança canônica"** = alteração de `sha256` do conteúdo do arquivo no `path`. (Simples, determinístico.)

- **Definition of Done (DoD):**
  - [ ] Estratégia escolhida e documentada.
  - [ ] Implementação produz eventos consistentes e auditáveis por mudança.

---

### [RESOLVED] Espelho do Vault ("quando desejado") — critério e feature flag

- **Priority:** P1
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** hidratação do contexto via Postgres e reindexação.

#### [DECISION] Solução escolhida

- **Flag por pasta** (whitelist) + override por frontmatter (`mirror: true`).
- **OFF** = ainda pode detectar mudança, mas faz **defer** (outbox) sem chunk/index.

- **Definition of Done (DoD):**
  - [ ] Contrato/documentação do comportamento on/off.
  - [ ] Fluxo de conflito documentado em `fluxos.md` (defer/retry/outbox).

---

### [RESOLVED] Estratégia de chunking (`NoteChunk`) + re-chunk

- **Priority:** P1
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** Vector Index e recuperação contextual (hidratando de chunks).

#### [DECISION] Solução escolhida

- **Chunking por seções Markdown** (títulos) com limite de tamanho (ex.: 1–2k tokens) e fallback por parágrafos.
- `sha256` por chunk + `chunkIndex`.
- **Rechunk: total por nota** no beta (mais simples, auditável). Otimiza depois.

- **Definition of Done (DoD):**
  - [ ] Estratégia documentada e implementada.
  - [ ] Auditoria/telemetria mínima para reindexações.

---

## P2 — Hardening e escolhas de superfície

### [RESOLVED] Enforce Telos singleton + migração

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 6.1). Em caso de conflito, a SPEC prevalece.

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** consistência da raiz ontológica; prevenção de multi‑`Telos` por erro.

#### [DECISION] Solução escolhida

- **ID fixo conhecido** (UUID constante) + impedir inserts com outro id.
- **Migração:** se existir 1, atualizar para id fixo; se existir >1, abortar migração e exigir intervenção manual (isso não deve "auto-resolver").

- **Definition of Done (DoD):**
  - [ ] Regra implementada no banco.
  - [ ] Migração/roteiro de migração definido (se já existir dado).
  - [ ] Contrato documentado em `contratos.md`.

---

### [RESOLVED] Definição de ciclos (cycle taxonomy) para TelosDeclaration

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 6.2). Em caso de conflito, a SPEC prevalece.

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** organização do histórico (não bloqueia funcionamento).

#### [DECISION] Solução escolhida

- `cycle` como **string livre**, mas com **convenção recomendada**:
  - `YYYY-Qn` ou `YYYY-MM`
  - Permitir "Primavera 2026" apenas como alias humano (não como classificação).
- **Não usar** `cycle` para lógica do sistema no beta (só organização).

- **Definition of Done (DoD):**
  - [ ] Padrão documentado.
  - [ ] Exemplo mínimo em `modelo-de-dados.md`/`fluxos.md`.

---

### [RESOLVED] Domínios/enums para `statusInternal` (RAW e canônico) e campos internos

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** validação de dados, UX de superfícies administrativas, consistência de estados.

#### [DECISION] Solução escolhida

- **Começar pequeno e extensível:**
  - `RawItem.statusInternal`: `captured | queued | clarified | placed | discarded`
  - `Poiesis.statusInternal`: `active | paused | done | archived`
  - `Energeia.statusInternal`: `open | done | canceled`
- **Transições permitidas explícitas** (documentar) + validação no backend.

- **Definition of Done (DoD):**
  - [ ] Enum/domínio definido e validado no banco.
  - [ ] Regras documentadas (transições e significado).

---

### [RESOLVED] Integridade referencial (FKs + `ON DELETE/ON UPDATE`) e cascatas

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** consistência do canônico sob deleções/tombstone e manutenção de relações.

#### [DECISION] Solução escolhida

- **Sem cascata de delete físico** (porque você está tombstoning).
- `ON UPDATE CASCADE` em IDs não faz sentido (UUID imutável).
- `ON DELETE RESTRICT` para impedir buracos; "deleção" é `deletedAt`.

- **Definition of Done (DoD):**
  - [ ] FKs e políticas definidas e implementadas.
  - [ ] Comportamento documentado (incluindo interações com `deletedAt`).

---

### [RESOLVED] Vector Index: provider/topologia (Vectorize vs pgvector vs híbrido)

- **Priority:** P2
- **Status:** Resolvido — decisão confirmada pelo DEV
- **Blocks:** implementação do plano de recuperação (indexação e query).

#### [DECISION] Solução escolhida

- **Provider:** **Cloudflare Vectorize**
- **Racional:** Já no ecossistema Cloudflare (D1, Workers). Serviço gerenciado, escalável, integrado com stack existente.
- **Namespaces:**
  - `vault-notes` — chunks de notas do Vault
  - `decisions` — registros de decisões/declarações
  - `raw` — (opcional, seletivo) capturas pré-ontológicas autorizadas
- **Metadata por vetor:** `path`, `noteId`, `chunkId`, `entityType`, `telosId`, `createdAt`

- **Definition of Done (DoD):**
  - [x] Provider decidido e documentado em `contratos.md`.
  - [ ] Pipeline de indexação validado (ao menos smoke test).

---

### [RESOLVED] RAW no índice semântico + salvaguardas

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** busca contextual que inclua capturas pré‑ontológicas.

#### [DECISION] Solução escolhida

- **Não indexar RAW por padrão.**
- Permitir "indexação seletiva" apenas quando `RawItem.statusInternal = queued` e após Clarify marcar "pode indexar".
- Namespace separado `raw` com filtros fortes e auditoria de inclusão/remoção.

- **Definition of Done (DoD):**
  - [ ] Decisão documentada e aplicada na indexação.
  - [ ] Auditoria registra inclusão/remoção de RAW no índice.

---

### [RESOLVED] Metadados mínimos obrigatórios por namespace

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** ranking e filtragem de resultados no plano de recuperação.

#### [DECISION] Solução escolhida

- **Para `vault-notes`:** `path`, `noteId`, `chunkId`, `sha256`, `updatedAt`
- **Para `decisions`:** `entityType`, `entityId`, `createdAt`
- **Para `raw`:** `rawItemId`, `createdAt`, `source`
- **Para todos:** `telosId` (sempre singleton, mas ajuda a manter contrato mental)

- **Definition of Done (DoD):**
  - [ ] Lista de metadata por namespace definida e implementada.

---

### [RESOLVED] Seleção de sessões Clarify (sem "rótulos para o SER")

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** ritual de Clarify e previsibilidade do processamento.

#### [DECISION] Solução escolhida

- **Seleção por tempo** (mais antigos primeiro) **ou** energia (limitar a N itens por sessão).
- **Apresentação:** lista simples + "vamos cuidar de 5 hoje" (sem badges).

- **Definition of Done (DoD):**
  - [ ] Critérios documentados e implementados no Lince.

---

### [RESOLVED] NocoDB: tabelas/visões expostas vs internas

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** edição administrativa e consultas factuais seguras.

#### [DECISION] Solução escolhida

- **Expor:** `Praxis/Ergon/Poiesis/Energeia`, e talvez `RawItem` (read-only).
- **Não expor:** `AuditEvent` (ou expor só via view agregada), `NoteChunk` (muito técnico).
- **`Note`** pode ser view read-only se necessário.

- **Definition of Done (DoD):**
  - [ ] Lista definida e refletida em views/permissões.

---

### [RESOLVED] Segurança (v0.1): usuários/papéis e permissões mínimas por superfície

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** isolamento entre SER, Lince e workers; prevenção de mutações acidentais.

#### [DECISION] Solução escolhida

- `lince_app` — read/write canônico + audit
- `worker_index` — read note/chunks + write index/outbox + audit
- `nocodb_admin` — read/write determinístico, sem acesso a segredos do índice
- `readonly` — para recuperação contextual em modo "só leitura"

- **Definition of Done (DoD):**
  - [ ] Roles/grants implementados e documentados.
  - [ ] Recuperação contextual com garantias read-only quando aplicável.

---

### [RESOLVED] Recuperação contextual: operações read-only e condições

- **Priority:** P2
- **Status:** Resolvido por convergência entre agentes
- **Blocks:** segurança e previsibilidade do plano de memória.

#### [DECISION] Solução escolhida

- **Default: read-only** em qualquer consulta que não seja comando imperativo explícito.
- **Só muda para write quando:**
  - Comando começa com verbo imperativo (criar/marcar/mover/excluir) **e**
  - Passa pelo protocolo de intenção + resolução + confirmação.

- **Definition of Done (DoD):**
  - [ ] Regra documentada e aplicada em runtime (guardrails).

---

## Resumo de Decisões

| Categoria | Total | Resolvidos | Pendentes |
| ----------- | ------- | ------------ | ---------- |
| **P0 — Bloqueadores** | 5 | 5 ✅ | 0 |
| **P1 — Infra memória** | 3 | 3 ✅ | 0 |
| **P2 — Hardening** | 12 | 12 ✅ | 0 |
| **TOTAL** | **20** | **20 ✅** | **0** |

**Status:** ✅ **Todas as decisões do TELOS v0.1 foram resolvidas.**

O documento está pronto para implementação. As decisões foram consolidadas por convergência entre agentes e validação do DEV.
