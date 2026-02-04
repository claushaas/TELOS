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

**Regra:** `Telos` é singleton (1 por instância). Implementação deve impedir criação de múltiplos registros.

### TelosDeclaration

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 3.2). Em caso de conflito, a SPEC prevalece.

Objetivo: registrar “efeitos do tempo sobre o SER” via escolhas declaradas, sem reificação.

Campos sugeridos (conceituais):

- `id` (uuid)
- `createdAt` (timestamptz)
- `telosId` (uuid) — aponta para o singleton (âncora)
- `statement` (text) — declaração curta do norte/compromisso (revisável)
- `cycle` (text, opcional) — ex.: “2026-Q1”, “primavera”, “ciclo atual”
- `isActive` (bool, opcional) — útil para marcar “declaração vigente” sem badges na UI do SER
- `supersedesId` (uuid, opcional) — encadeamento de revisões (histórico)
- `auditEventId` (uuid, opcional) — amarra à auditoria
- `source` (text, opcional) — ex.: `clarify-session`, `lince-chat`

Regras:

- Nunca inferir `statement`; sempre criado por comando/Clarify explícito.
- O sistema funciona sem `TelosDeclaration` (histórico é opcional).

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

## Auditoria (mínimo)

[SOURCE] SPEC v0.1 — “Auditabilidade acima de conveniência” + fluxos canônicos (registro de ação e trilha).

> Nota: nomes reais (snake_case/camelCase) continuam pendentes; aqui é o modelo conceitual.

### AuditEvent

- `id` (uuid)
- `createdAt` (timestamptz)
- `actor` (ex.: `SER|LINCE|WORKER`)
- `source` (ex.: `lince-chat|clarify-session|vault-sync|api`)
- `action` (ex.: `CREATE|UPDATE|DELETE|PROMOTE|ATTACH|COMPLETE`)
- `entityType`
- `entityId` (uuid)
- `before` (jsonb)
- `after` (jsonb)
- `reason` (text, opcional)
- `correlationId` (uuid, opcional)
- `rawItemId` (uuid, opcional)
- `noteId` (uuid, opcional)
- `path` (text, opcional)

## Promoção RAW → Canônico (rastreabilidade)

[SOURCE] SPEC v0.1 — RAW + Clarify + “Auditabilidade acima de conveniência”.

### RawPromotion (opcional, recomendado)

Se a implementação não quiser incluir `raw_item_id` diretamente nas entidades TELOS, criar uma tabela de mapeamento:

- `id` (uuid)
- `rawItemId` (uuid)
- `entityType`
- `entityId` (uuid)
- `createdAt` (timestamptz)
- `auditEventId` (uuid)

## Reversibilidade mínima (tombstone)

[SOURCE] SPEC v0.1 — execução sob comando explícito (preferência por reversibilidade e logs).

Para entidades principais (ao menos Poiesis e Energeia), preferir deleção lógica:

- `deletedAt` (timestamptz, opcional)

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
- **Auditoria**: estrutura mínima do log (ator, ação, before/after, referências), incluindo `correlationId` e política de tombstone/undo.
