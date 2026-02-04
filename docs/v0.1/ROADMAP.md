# ROADMAP — TELOS Beta v0.1

> **Status:** Planejado  
> **Horizonte:** Implementação incremental (fases sequenciais)  
> **Fonte de verdade:** [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md)  
> **Decisões consolidadas (P0–P2):** [`pendencias.md`](./pendencias.md)

---

## 1. Scope Framing (In/Out)

### In Scope (v0.1)

[SOURCE] [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Seção "Estado do projeto"

- Ontologia canônica completa: Telos (singleton) → Praxis → Ergon → Poiesis → Energeia
- Pipeline RAW → Clarify → Canônico com rastreabilidade
- Espelho do Vault (Obsidian) com chunking e indexação Vectorize
- Interface cognitiva via Lince (captura, clarify, execução determinística)
- Auditoria mínima com tombstone e AuditEvent
- Superfície determinística NocoDB para Jardineiro

### Out of Scope (v0.1)

[SOURCE] [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Seção "Fora do Escopo v0.1"

- Multi-tenant
- Colaboração entre usuários
- Versionamento histórico profundo
- UI própria do SER (além de Lince + NocoDB + Vault)
- Sincronização bidirecional automática Vault↔Postgres

---

## 2. Fases de Implementação

Cada fase constrói sobre a anterior. Não pular fases sem validação explícita.

> **Nota sobre guardrails:** Itens críticos de integridade (conflito checkbox, anti-reificação linguística) foram antecipados para a **Fase 3** mesmo que tecnicamente pudessem vir depois. Isso evita que uma versão intermediária "vaze" comportamentos indesejados.

---

### Fase 1: Fundação Canônica (Postgres)

**Theme:** Estabelecer a fonte única de verdade com constraints e audit trail base.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 1.1 | Schema base: Telos, Praxis, Ergon, Poiesis, Energeia | DDL aplicado, FKs e constraints ativos | [`modelo-de-dados.md`](./modelo-de-dados.md) — Entidades canônicas |
| 1.2 | Enforcement Telos singleton | Constraint + UUID fixo, migração testada | [`pendencias.md`](./pendencias.md) — Item 9 |
| 1.3 | Campos derivados (telosId, praxisId em cascata) | Trigger ou API recalcula, input direto ignorado | [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Seção 8.2 |
| 1.4 | Tabela AuditEvent (esqueleto) | Estrutura criada, índices em correlationId e timestamps | [`pendencias.md`](./pendencias.md) — Item 4 |
| 1.5 | Tombstone base (`deletedAt`) | Coluna adicionada a Poiesis e Energeia | [`pendencias.md`](./pendencias.md) — Item 4 |

**Milestone Exit Criteria:**

- [ ] Migrations idempotentes rodando do zero
- [ ] Teste: inserir segundo Telos falha (constraint)
- [ ] Teste: deleção lógica funciona (tombstone)

**Status:** 🔲 Não iniciado

---

### Fase 2: Captura e Solo Fértil (RAW)

**Theme:** Permitir descarga mental sem obrigação de forma.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 2.1 | Tabela RawItem | DDL com campos mínimos: `rawText`, `statusInternal`, `source` | [`modelo-de-dados.md`](./modelo-de-dados.md) — RawItem |
| 2.2 | Enum statusInternal (RAW) | Valores: `captured \| queued \| clarified \| placed \| discarded` | [`pendencias.md`](./pendencias.md) — Item 11 |
| 2.3 | Pipeline de captura via Lince | Comando "capturar" cria RawItem, retorna confirmação | [`fluxos.md`](./fluxos.md) — Fluxo 1 (Captura) |
| 2.4 | Agendamento soft de Clarify | Campo `scheduledClarifyAt`, consulta por itens pendentes | [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Seção 7.4 |

**Milestone Exit Criteria:**

- [ ] Lince captura input e cria RawItem
- [ ] RawItem preserva texto original imutável
- [ ] Lista de pendentes retorna itens por tempo

**Status:** 🔲 Não iniciado

---

### Fase 3: Fluxo Clarify (Processamento Conjunto)

**Theme:** Transformar intenção em estrutura via diálogo SER+Lince.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 3.1 | Tabela RawPromotion (rastreabilidade) | 1 RawItem → N entidades mapeáveis | [`pendencias.md`](./pendencias.md) — Item 5 |
| 3.2 | Protocolo de intenção explícita | Comandos imperativos detectados, confirmação solicitada | [`pendencias.md`](./pendencias.md) — Item 1 |
| 3.3 | Pipeline 3 estágios de resolução | ID/path → busca determinística → candidatos com ranking | [`pendencias.md`](./pendencias.md) — Item 2 |
| 3.4 | Confirmação dupla (alto impacto) | Deleção, massa, promoção RAW→N entidades exigem dupla confirmação | [`pendencias.md`](./pendencias.md) — Item 1 |
| 3.5 | Perguntas mínimas de Clarify | Lince pergunta: contínuo vs finito? resultado? prazo? onde encaixa? | [`fluxos.md`](./fluxos.md) — Seção 2 (Perguntas mínimas) |
| 3.6 | Promoção RAW → Canônico | Criar entidade, mapear em RawPromotion, atualizar status | [`fluxos.md`](./fluxos.md) — Fluxo 2 (Clarify) |
| 3.7 | Decomposição (1 raw → N) | RawItem gera múltiplas entidades, todas rastreáveis | [`pendencias.md`](./pendencias.md) — Item 5 |
| 3.8 | Descarte com registro | RawItem pode ser descartado, auditado, sem culpa | [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Seção 6.3 |
| 3.9 | Política de conflito checkbox | Checkbox nunca é canal autorizado; gera RawItem para Clarify | [`pendencias.md`](./pendencias.md) — Item 3 |
| 3.10 | Linguagem de agência | Lince usa: "você declarou", "você escolheu", "quer revisar?" | [`contratos.md`](./contratos.md) — Anti-Reificação do SER |
| 3.11 | Recusa de reificação | Lince recusa "defina meu telos" com explicação | [`contratos.md`](./contratos.md) — Anti-Reificação do SER |

**Milestone Exit Criteria:**

- [ ] Sessão Clarify completa: entrada → perguntas → promoção → auditoria
- [ ] Resolução de "tarefa X" funciona com confirmação quando ambíguo
- [ ] Decomposição de 1 raw em múltiplas energias rastreável
- [ ] Checkbox manual no Obsidian gera RawItem (não altera canônico)
- [ ] Lince nunca diz "seu telos é X" nem aceita comandos de reificação

**Status:** 🔲 Não iniciado

---

### Fase 4: Memória Textual (Vault + Vectorize)

**Theme:** Espelhar, fragmentar e indexar o conhecimento do Vault.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 4.1 | Tabelas Note e NoteChunk | DDL com `path`, `sha256`, `chunkIndex` | [`modelo-de-dados.md`](./modelo-de-dados.md) — Espelho do Vault |
| 4.2 | Detecção de mudanças no Vault | Watcher fs events + fallback polling por hash | [`pendencias.md`](./pendencias.md) — Item 6 |
| 4.3 | Feature flag de espelhamento | Flag por pasta (whitelist) + override frontmatter | [`pendencias.md`](./pendencias.md) — Item 7 |
| 4.4 | Chunking por seções Markdown | Delimitador: títulos (#, ##), limite de tokens, fallback parágrafos | [`pendencias.md`](./pendencias.md) — Item 8 |
| 4.5 | Re-chunk total por nota | Mudança de nota → recria todos os chunks | [`pendencias.md`](./pendencias.md) — Item 8 |
| 4.6 | Integração Cloudflare Vectorize | Index criado, namespaces definidos (`vault-notes`, `decisions`) | [`pendencias.md`](./pendencias.md) — Item 13 |
| 4.7 | Metadata nos vetores | `path`, `noteId`, `chunkId`, `entityType`, `telosId`, `createdAt` | [`pendencias.md`](./pendencias.md) — Item 15 |
| 4.8 | Hidratação pós-busca | Vector retorna chunkIds → Lince busca texto em NoteChunk | [`fluxos.md`](./fluxos.md) — Fluxo 4 (Recuperação contextual) |
| 4.9 | Smoke test de recuperação | Dada 1 nota com 3 chunks, query retorna chunk esperado + hidratação bate com sha256 | [RISK] Pipeline vetorial validado antes de uso produtivo |

**Milestone Exit Criteria:**

- [ ] Mudança no Vault detectada e espelhada em < 5 minutos
- [ ] Query semântica retorna chunks relevantes
- [ ] Hidratação traz texto completo do Postgres
- [ ] Smoke test passa: 1 nota → 3 chunks → query → chunk correto → sha256 bate

**Status:** 🔲 Não iniciado

---

### Fase 5: Superfícies e Segurança

**Theme:** Ferramentas do Jardineiro e guardrails de acesso.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 5.1 | Configuração NocoDB | Conexão Postgres, views de Praxis/Ergon/Poiesis/Energeia | [`pendencias.md`](./pendencias.md) — Item 17 |
| 5.2 | Views restritas (NocoDB) | AuditEvent e NoteChunk não expostos (ou read-only) | [`pendencias.md`](./pendencias.md) — Item 17 |
| 5.3 | Roles de banco | `lince_app`, `worker_index`, `nocodb_admin`, `readonly` | [`pendencias.md`](./pendencias.md) — Item 18 |
| 5.4 | Política read-only default | Lince só escreve após comando imperativo + protocolo | [`pendencias.md`](./pendencias.md) — Item 19 |

**Milestone Exit Criteria:**

- [ ] NocoDB acessível para edição de entidades canônicas
- [ ] Roles com permissões mínimas ativas
- [ ] Política de execução via Lince (não direta) documentada e aplicada

**Status:** 🔲 Não iniciado

---

### Fase 6: Temporalidade (TelosDeclaration)

**Theme:** Registrar escolhas do SER ao longo do tempo, sem transformar em identidade.

[SOURCE] [`pendencias.md`](./pendencias.md) — Itens 9, 10; [`contratos.md`](./contratos.md) — Anti-Reificação

> **Nota:** Guardrails de linguagem (anti-reificação) já implementados na Fase 3. Esta fase adiciona a capacidade de registrar declarações temporais explicitamente.

| Etapa | Descrição | Critério de Conclusão | Decisões [SOURCE] |
| ----- | --------- | -------------------- | ----------------- |
| 6.1 | Tabela TelosDeclaration | Campos: `statement`, `cycle`, `isActive`, `supersedesId` | [`modelo-de-dados.md`](./modelo-de-dados.md) — TelosDeclaration |
| 6.2 | Fluxo de declaração temporal | SER solicita → Lince confirma → cria registro + auditoria | [`fluxos.md`](./fluxos.md) — Fluxo 6 (Declaração temporal) |

**Milestone Exit Criteria:**

- [ ] Declaração criada com `correlationId` amarrando à auditoria
- [ ] Ciclo documentado como string livre (ex: "2026-Q1")
- [ ] Histórico de declarações acessível para consulta (não exibido como identidade)

**Status:** 🔲 Não iniciado

---

## 3. Dependências

| Dependência | Fase que precisa | Tipo |
| ----------- | --------------- | ----- |
| Postgres (local ou D1) | Fase 1 | Infraestrutura |
| Acesso Cloudflare Vectorize | Fase 4 | Infraestrutura |
| Vault Obsidian acessível | Fase 4 | Infraestrutura |
| NocoDB deployado | Fase 5 | Ferramenta |
| openclaw/Lince configurado | Todas as fases | Runtime |

---

## 4. Riscos

| Risco | Probabilidade | Impacto | Mitigação |
| ----- | ------------- | ------- | --------- |
| Drift entre Vault e Postgres | Média | Alto | Polling + hash, auditoria de sync |
| Ambiguidade em resolução de alvos | Alta (início) | Médio | Sempre confirmação quando não há match único forte |
| Complexidade do Vectorize | Média | Médio | Smoke test obrigatório antes de considerar fase 4 completa |
| Over-engineering em auditoria | Média | Baixo | Começar com campos obrigatórios mínimos, estender depois |

---

## 5. Status Atual

| Fase | Status | Bloqueios |
| ----- | ------- | --------- |
| 1. Fundação Canônica | 🔲 Não iniciado | Nenhum |
| 2. Captura e RAW | 🔲 Não iniciado | Depende da Fase 1 |
| 3. Fluxo Clarify | 🔲 Não iniciado | Depende da Fase 2 |
| 4. Memória Textual | 🔲 Não iniciado | Depende da Fase 1; Vectorize |
| 5. Superfícies e Segurança | 🔲 Não iniciado | Depende da Fase 1 |
| 6. Anti-Reificação | 🔲 Não iniciado | Depende da Fase 1 |

---

## 6. Links

### Documentos Canônicos (v0.1)

- [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Especificação técnica
- [`contratos.md`](./contratos.md) — Contratos e invariantes
- [`fluxos.md`](./fluxos.md) — Fluxos canônicos
- [`modelo-de-dados.md`](./modelo-de-dados.md) — Entidades e campos
- [`glossario.md`](./glossario.md) — Vocabulário
- [`pendencias.md`](./pendencias.md) — Decisões resolvidas

### Contexto Filosófico (fora de v0.1)

- [`../../README.md`](../../README.md) — Visão geral TELOS
- [`../../WHY_TELOS.md`](../../WHY_TELOS.md) — Fundamentação filosófica
- [`../../anti-TELOS.md`](../../anti-TELOS.md) — Non-goals

---

## 7. Notas de Manutenção

- **Drift é defeito:** Este roadmap deve ser atualizado quando:
  - Uma fase é concluída (marcar ✅ e data)
  - Uma decisão muda (atualizar referência para [`pendencias.md`](./pendencias.md))
  - Um risco se materializa (documentar mitigação aplicada)

- **Exploratório:** Fases 4-6 podem ter ordem trocada entre si se necessidade operacional mudar, mas Fases 1-3 são sequenciais estritas. Guardrails críticos (checkbox, anti-reificação) foram antecipados para Fase 3.

- **[SOURCE]** e **[PENDING]:** Novos itens adicionados ao roadmap devem usar as convenções de marcação da v0.1.
