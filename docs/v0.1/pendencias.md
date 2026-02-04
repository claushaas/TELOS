# Pendências e Decisões Abertas — TELOS v0.1

> [SOURCE] Este arquivo existe para registrar o que a SPEC v0.1 (ver [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md)) **não fixa** (ainda), evitando invenção e drift.

## P0 — Bloqueadores (segurança + auditabilidade)

### [PENDING] Protocolo de intenção explícita + confirmação (Lince executor)

- **Priority:** P0
- **Blocks:** execução determinística via Lince (CRUD), especialmente ações irreversíveis/alto impacto.
- **Decision needed:** quais operações exigem confirmação (simples vs dupla) e qual é o “canal autorizado” para execução.
- **Definition of Done (DoD):**
  - Contrato documentado em `contratos.md`.
  - Fluxo documentado em `fluxos.md`.
  - Implementação registra auditoria antes/depois e recusa executar fora das regras.

### [PENDING] Resolução de referência (alvos humanos / “tarefa X”)

- **Priority:** P0
- **Blocks:** comandos do SER que referenciam entidades por nome/trecho/path (ex.: completar Energeia, criar Poiesis dentro de Ergon).
- **Decision needed:** algoritmo de resolução (determinístico → candidatos → sem candidatos) e regra de ambiguidade/confirmação.
- **Definition of Done (DoD):**
  - Contrato documentado em `contratos.md` (incluindo definição de `human_ref`).
  - Fluxo “resolução de alvo + confirmação” em `fluxos.md`.
  - Auditoria inclui `human_ref`/candidatos/decisão (quando aplicável).

### [PENDING] Conflito: checkbox do Obsidian ↔ status canônico (Energeia)

- **Priority:** P0
- **Blocks:** sincronização Vault↔Postgres e consistência de status de Energeia.
- **Decision needed:** quando (e como) uma mudança de checkbox pode ser tratada como intenção explícita; como registrar e resolver conflitos.
- **Definition of Done (DoD):**
  - Contrato documentado em `contratos.md`.
  - Fluxo de conflito documentado em `fluxos.md`.
  - Auditoria registra conflitos e resolução.

### [PENDING] Auditoria mínima + reversibilidade (AuditEvent, `auditRef`, undo/tombstone)

- **Priority:** P0
- **Blocks:** qualquer mutação canônica (promoção RAW, CRUD, sync, indexação).
- **Decision needed:** esquema mínimo de evento (campos), política de tombstone (`deletedAt`) e regra de undo (janela vs sempre, e para quais ações).
- **Definition of Done (DoD):**
  - Modelo conceitual em `modelo-de-dados.md` (AuditEvent e campos).
  - Contrato de audit trail em `contratos.md`.
  - Implementação registra eventos com `correlationId` e suporta tombstone/undo conforme regra definida.

### [PENDING] Promoção RAW → Canônico com rastreabilidade

- **Priority:** P0
- **Blocks:** sessões Clarify e qualquer promoção/anexo/decomposição que altere ontologia.
- **Decision needed:** como preservar vínculo (`raw_item_id` na entidade vs tabela de mapeamento) e como registrar decomposição (1 raw → N entidades).
- **Definition of Done (DoD):**
  - Contrato em `contratos.md`.
  - Modelo em `modelo-de-dados.md` (ex.: `RawPromotion` opcional).
  - Auditoria amarra RawItem ↔ entidade(s) canônica(s).

## P1 — Infra do plano de memória (Vault / espelho / chunking)

### [PENDING] Detecção de mudanças no Vault + gatilho canônico

- **Priority:** P1
- **Blocks:** espelho do Vault, indexação e recuperação contextual confiável.
- **Decision needed:** mecanismo de detecção (watcher/plugin/polling/git/etc.) e o que é considerado “mudança canônica”.
- **Definition of Done (DoD):**
  - Estratégia escolhida e documentada.
  - Implementação produz eventos consistentes e auditáveis por mudança.

### [PENDING] Espelho do Vault (“quando desejado”) — critério e feature flag

- **Priority:** P1
- **Blocks:** hidratação do contexto via Postgres e reindexação.
- **Decision needed:** critério/flag por ambiente, e o comportamento quando estiver desligado.
- **Definition of Done (DoD):**
  - Contrato/documentação do comportamento on/off.
  - Fluxo de conflito documentado em `fluxos.md` (defer/retry/outbox).

### [PENDING] Estratégia de chunking (`NoteChunk`) + re-chunk

- **Priority:** P1
- **Blocks:** Vector Index e recuperação contextual (hidratando de chunks).
- **Decision needed:** tamanho/delimitadores/hashing e política de re-chunk (parcial vs total).
- **Definition of Done (DoD):**
  - Estratégia documentada e implementada.
  - Auditoria/telemetria mínima para reindexações.

## P2 — Hardening e escolhas de superfície

### [PENDING] Enforce Telos singleton + migração

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 6.1). Em caso de conflito, a SPEC prevalece.

- **Priority:** P2
- **Blocks:** consistência da raiz ontológica; prevenção de multi‑`Telos` por erro.
- **Decision needed:** mecanismo de enforcement (id fixo vs constraint) e como tratar dados existentes.
- **Definition of Done (DoD):**
  - Regra implementada no banco.
  - Migração/roteiro de migração definido (se já existir dado).
  - Contrato documentado em `contratos.md`.

### [PENDING] Definição de ciclos (cycle taxonomy) para TelosDeclaration

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 6.2). Em caso de conflito, a SPEC prevalece.

- **Priority:** P2
- **Blocks:** organização do histórico (não bloqueia funcionamento).
- **Decision needed:** como nomear ciclos (data, estações, trimestres, etc.) sem virar rótulo.
- **Definition of Done (DoD):**
  - Padrão documentado.
  - Exemplo mínimo em `modelo-de-dados.md`/`fluxos.md`.

### [PENDING] Domínios/enums para `statusInternal` (RAW e canônico) e campos internos

- **Priority:** P2
- **Blocks:** validação de dados, UX de superfícies administrativas, consistência de estados.
- **Decision needed:** quais estados existem por entidade e quais transições são válidas.
- **Definition of Done (DoD):**
  - Enum/domínio definido e validado no banco.
  - Regras documentadas (transições e significado).

### [PENDING] Integridade referencial (FKs + `ON DELETE/ON UPDATE`) e cascatas

- **Priority:** P2
- **Blocks:** consistência do canônico sob deleções/tombstone e manutenção de relações.
- **Decision needed:** políticas de cascata vs restrição por entidade, compatíveis com tombstone.
- **Definition of Done (DoD):**
  - FKs e políticas definidas e implementadas.
  - Comportamento documentado (incluindo interações com `deletedAt`).

### [PENDING] Vector Index: provider/topologia (Vectorize vs pgvector vs híbrido)

- **Priority:** P2
- **Blocks:** implementação do plano de recuperação (indexação e query).
- **Decision needed:** provider, ambientes/namespaces e como isso afeta hidratação/metadata.
- **Definition of Done (DoD):**
  - Provider decidido e documentado em `contratos.md`.
  - Pipeline de indexação validado (ao menos smoke test).

### [PENDING] RAW no índice semântico + salvaguardas

- **Priority:** P2
- **Blocks:** busca contextual que inclua capturas pré‑ontológicas.
- **Decision needed:** se RAW é indexado, em qual namespace, e quais regras de privacidade/auditoria se aplicam.
- **Definition of Done (DoD):**
  - Decisão documentada e aplicada na indexação.
  - Auditoria registra inclusão/remoção de RAW no índice.

### [PENDING] Metadados mínimos obrigatórios por namespace

- **Priority:** P2
- **Blocks:** ranking e filtragem de resultados no plano de recuperação.
- **Decision needed:** campos obrigatórios por namespace (além de `path/entityType/telosId/...`) e suas garantias.
- **Definition of Done (DoD):**
  - Lista de metadata por namespace definida e implementada.

### [PENDING] Seleção de sessões Clarify (sem “rótulos para o SER”)

- **Priority:** P2
- **Blocks:** ritual de Clarify e previsibilidade do processamento.
- **Decision needed:** critérios de seleção (tempo/energia/área) e como apresentar sem badges/labels.
- **Definition of Done (DoD):**
  - Critérios documentados e implementados no Lince.

### [PENDING] NocoDB: tabelas/visões expostas vs internas

- **Priority:** P2
- **Blocks:** edição administrativa e consultas factuais seguras.
- **Decision needed:** quais entidades aparecem no NocoDB e quais ficam internas (ex.: auditoria).
- **Definition of Done (DoD):**
  - Lista definida e refletida em views/permissões.

### [PENDING] Segurança (v0.1): usuários/papéis e permissões mínimas por superfície

- **Priority:** P2
- **Blocks:** isolamento entre SER, Lince e workers; prevenção de mutações acidentais.
- **Decision needed:** quais papéis existem (mesmo single-user) e quais permissões cada um tem.
- **Definition of Done (DoD):**
  - Roles/grants implementados e documentados.
  - Recuperação contextual com garantias read-only quando aplicável.

### [PENDING] Recuperação contextual: operações read-only e condições

- **Priority:** P2
- **Blocks:** segurança e previsibilidade do plano de memória.
- **Decision needed:** quando o Lince pode apenas ler vs escrever, por superfície e por comando.
- **Definition of Done (DoD):**
  - Regra documentada e aplicada em runtime (guardrails).
