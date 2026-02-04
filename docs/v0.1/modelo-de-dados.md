# Modelo de Dados — TELOS v0.1

> [SOURCE] Derivado de [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) (seções 2, 4, 6, 8, 9, 10, 12). Em caso de conflito, a SPEC prevalece.

## Escopo

Este documento descreve **entidades e campos mínimos** mencionados na SPEC v0.1, mais as **regras de unicidade e derivação** que garantem consistência.

## Relações canônicas (ontologia)

```text
Telos
 └─ Praxis
     └─ Ergon
         └─ Poiesis
             └─ Energeia
```

## Entidades (camada canônica)

### Telos

- `id` (uuid)
- `slug`
- `title`
- `description`

### Praxis

- `id` (uuid)
- `telosId` (uuid)
- `slug`
- `title`

### Ergon

- `id` (uuid)
- `praxisId` (uuid)
- `telosId` (uuid, **derivado**)
- `slug`
- `title`

### Poiesis

- `id` (uuid)
- `ergonId` (uuid)
- `praxisId` (uuid, **derivado**)
- `telosId` (uuid, **derivado**)
- `slug`
- `title`
- `statusInternal` (interno)

### Energeia

- `id` (uuid)
- `poiesisId` (uuid)
- `slug`
- `title`
- `statusInternal` (interno)
- `completedAt` (timestamptz, quando aplicável)

## Entidades (Plano RAW — pré‑ontológico)

### RawItem

Campos mínimos sugeridos na SPEC:

- `id` (uuid)
- `createdAt` (timestamptz)
- `source` (ex.: `lince-chat`, `voice`, `mobile`)
- `rawText` (imutável)
- `normalizedText` (opcional; nunca substitui `rawText`)
- `statusInternal` (interno; ex.: `captured|queued|clarified|placed|discarded`)
- `scheduledClarifyAt` (opcional)
- `placedEntityType` + `placedEntityId` (quando promovido)
- `auditRef` (opcional)

## Entidades (Espelho do Vault — memória textual)

### Note

- `id`
- `path` (unique)
- `content`
- `sha256`
- `updatedAt`

### NoteChunk

- `id`
- `noteId`
- `chunkIndex`
- `content`
- `sha256`
- `updatedAt`

## Regras de unicidade (por escopo)

[SOURCE] SPEC v0.1 — “Unicidade por escopo”.

- **Praxis:** `(telosId, slug)`
- **Ergon:** `(praxisId, slug)`
- **Poiesis:** `(ergonId, slug)`
- **Energeia:** `(poiesisId, slug)`

## Campos derivados (regra de consistência)

[SOURCE] SPEC v0.1 — “Regra de consistência”.

- Campos derivados (ex.: `telosId`, `praxisId` em camadas inferiores) são **recalculados pela API/Processador** e devem **ignorar input direto**.

## [PENDING] Definições necessárias para implementação

- **Nomes reais de tabelas/colunas** (snake_case vs camelCase), e convenções de schema.
- **Domínios/enums** para `statusInternal` (e outros campos internos citados na SPEC como “podem existir”).
- **Políticas de FK** (`ON DELETE` / `ON UPDATE`) e regras de cascata.
- **Auditoria**: estrutura mínima do log (ator, ação, before/after, referências) para cumprir “auditabilidade acima de conveniência”.
