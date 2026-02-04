# Contratos e Invariantes — TELOS v0.1

> [SOURCE] Derivado de [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md). Em caso de conflito, a SPEC prevalece.

## Princípios contratuais (o que não pode quebrar)

[SOURCE] SPEC v0.1 — “Princípios Canônicos”.

- **Postgres é a fonte única de verdade** para ontologia, relações e estados canônicos.
- **RAW é pré‑ontológico**: captura bruta não compete com a ontologia.
- **Separação Memória vs Processador**:
  - Memória preserva e recupera contexto.
  - Processador decide encaixe, executa mudanças, registra ação.
- **Ação só sob intenção explícita**: o Lince não faz alterações silenciosas/inferidas.
- **Auditabilidade acima de conveniência**: sincronização/promoção/execução precisam de trilha.

## Superfícies (papéis e prioridade)

[SOURCE] SPEC v0.1 — “Separação entre determinístico e contextual”.

- **NocoDB**: superfície preferencial para consultas factuais e edição administrativa (determinístico).
- **Lince**: superfície preferencial para busca conceitual/contextual (semântico).
- **Lince pode executar CRUD determinístico** apenas sob comando explícito.

## Identidade: `id`, `slug`, `path`, `canonical_key`

[SOURCE] SPEC v0.1 — “Identidade, Slug e Caminho”.

- `id`: UUID canônico, imutável.
- `slug`: identificador local ao escopo (sem `/`).
- `path`: caminho físico no Vault.
- `canonical_key` (opcional): identificador composto legível.

### Unicidade por escopo

[SOURCE] SPEC v0.1 — “Unicidade por escopo”.

- Praxis: `(telosId, slug)`
- Ergon: `(praxisId, slug)`
- Poiesis: `(ergonId, slug)`
- Energeia: `(poiesisId, slug)`

## Contrato mínimo com o Obsidian (Vault)

[SOURCE] SPEC v0.1 — “Frontmatter Obsidian (Contrato mínimo)”.

### Poiesis (frontmatter)

```yaml
telos_id: uuid
praxis_id: uuid
ergon_id: uuid
poiesis_id: uuid
slug: rotina-inicial
```

### Energeia (visual / checklist)

```md
- [ ] Revisar plano semanal <!-- energeia_id: uuid -->
```

## Contrato do Plano de Recuperação (Vector Index)

[SOURCE] SPEC v0.1 — “Plano de Recuperação (Vector Index)”.

- Índice por ambiente (ex.: `telos-beta`).
- Namespaces por tipo (ex.: `vault-notes`, `decisions`, `raw` opcional).
- Vetores apontam para `NoteChunk.id` (ou `RawItem.id` se indexado).
- Metadata mínima (quando aplicável): `path`, `entityType`, `telosId`, `praxisId`, `ergonId`, `poiesisId`.

## UI do SER: conformidade sem rótulos

[SOURCE] SPEC v0.1 — “Conformidade TELOS — Rótulos e Estado”.

- Campos operacionais (`statusInternal`, `priorityInternal`, `tagsInternal`) **podem existir** como dados.
- A **UI do SER não deve exibir labels/badges**.

## [PENDING] Contratos que precisam ser fixados para evitar drift

- **Audit trail**: formato de eventos e como referenciar (`auditRef`), incluindo reversibilidade/undo quando aplicável.
- **Vector Index provider**: decisão entre “Vectorize/pgvector” e como isso impacta ambientes, namespaces e hidratação.
- **Espelho do Vault “quando desejado”**: critério/feature flag para habilitar armazenamento/espelhamento no Postgres.
