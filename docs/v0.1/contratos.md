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

## Anti‑Reificação do SER (Regra Fundamental)

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 2.1). Em caso de conflito, a SPEC prevalece.

- Proibido: frases normativas/identitárias do tipo **“seu telos é X”**, **“você é X”**, **“o sistema concluiu quem você é”**.
- Permitido: linguagem de agência e temporalidade:
  - “você **declarou** X em {data}”
  - “você **escolheu** X para este ciclo”
  - “quer **revisar** X?”
  - “hipótese: suas ações recentes **parecem** (des)alinhadas com X” (sempre como hipótese e com referências)
- O Lince deve recusar reificação mesmo quando solicitado (“defina meu telos”).

## Superfícies (papéis e prioridade)

[SOURCE] SPEC v0.1 — “Separação entre determinístico e contextual”.

- **NocoDB**: superfície preferencial para consultas factuais e edição administrativa (determinístico).
- **Lince**: superfície preferencial para busca conceitual/contextual (semântico).
- **Lince pode executar CRUD determinístico** apenas sob comando explícito.

## Telos no Postgres: Âncora Singleton (não identidade)

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 2.2). Em caso de conflito, a SPEC prevalece.

- Existe **exatamente 1** registro de `Telos` por instância TELOS.
- `Telos` é **âncora do grafo**, não cadastro do SER.
- A existência/edição de `Telos` não deve ser exibida como “perfil” na UI do SER.
- Qualquer histórico de norte deve ser modelado como **declarações temporais** (ex.: `TelosDeclaration`), não como atributo permanente.
- O sistema funciona mesmo sem histórico (declarações são opcionais).

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

## Contrato de Resolução de Referência (alvos humanos)

**Objetivo:** suportar comandos como “marque a tarefa X como concluída” de forma segura e auditável.

- Definir o conceito de **referência humana** (`human_ref`): string fornecida pelo SER (nome, trecho, path, etc.).
- Definir algoritmo de resolução em 3 estágios:
  1) **Resolução determinística** (id explícito / path exato / `canonical_key`) → pode executar.
  2) **Resolução com candidatos** (busca + ranking) → exige confirmação se houver >1 candidato, baixa confiança ou ambiguidade.
  3) **Sem candidatos** → pedir mais contexto (não inventar).
- Política de confirmação:
  - Operação **irreversível** ou com alto impacto (muitas entidades) → confirmação dupla.
  - Ambiguidade na resolução → confirmação obrigatória antes de executar.

## Contrato de Conflito: Obsidian Checklist vs Postgres

**Regra:** status canônico de Energeia vive no Postgres. Checkbox no Obsidian é **projeção/visual** e pode ser atualizado pelo sistema.

**Conflitos (checkbox → canônico):**

- Se checkbox mudar e houver `energeia_id`, o sistema só pode tratar como **intenção explícita** se a mudança ocorrer por um **canal autorizado** (ex.: comando do Lince / ferramenta autorizada). Caso contrário, entra como RAW/pendência de Clarify.
- Se não houver `energeia_id`, checkbox não pode alterar canônico; vira sinal a ser processado (RAW/Clarify).

## [PENDING] Contratos que precisam ser fixados para evitar drift

- **Audit trail (evento mínimo)**: formato de eventos e como referenciar (`auditRef`).
  - Campos mínimos: `id`, `createdAt`, `actor`, `source`, `action`, `entityType`, `entityId`, `before`, `after`, `reason`, `correlationId`, `rawItemId?`, `noteId?`, `path?`.
  - Reversibilidade mínima:
    - Soft delete via `deletedAt`/tombstone (preferível no beta).
    - “Undo window” opcional (ex.: 24h) ou “undo sempre” para ações simples.
- **Contrato de Promoção RAW → Canônico**:
  - Campos obrigatórios para criar cada entidade (Telos/Praxis/Ergon/Poiesis/Energeia).
  - Preservar vínculo de origem: `raw_item_id` na entidade criada **ou** tabela de mapeamento.
  - Registrar decomposição (1 RawItem → N entidades).
- **Enforcement do singleton Telos:** mecanismo (id fixo vs constraint) e regras de migração (se já existir dado).
- **Vector Index provider**: decisão entre “Vectorize/pgvector” e como isso impacta ambientes, namespaces e hidratação.
- **Espelho do Vault “quando desejado”**: critério/feature flag para habilitar armazenamento/espelhamento no Postgres.
