<!-- markdownlint-disable MD036 -->
# TELOS — Especificação Técnica v0.1 (Beta)

> **Status:** Beta operacional (embrião)
>
> **Objetivo:** Definir a arquitetura mínima, consistente e auditável do TELOS rodando sobre **Postgres + Obsidian (Vault) + NocoDB + Vector Index (Vectorize/pgvector) + openclaw (Lince)**, com um fluxo humano de **captura → esclarecimento → assentamento** que respeita o SER, e um contrato explícito entre **memória** e **processamento**.

---

## 1. Princípios Canônicos

1. **Postgres é a fonte única de verdade.** Ontologia, relações e estados canônicos vivem no banco.
2. **RAW é pré‑ontológico.** Capturas brutas não competem com a ontologia TELOS; são solo, não fruto.
3. **Anti‑reificação do SER.** O TELOS não declara “o seu telos é X”. O sistema registra apenas **declarações/compromissos datados** escolhidos pelo SER (revisáveis), e nunca os usa como atributo identitário.
4. **Separação entre Memória e Processador.**
   - **Memória:** preserva o que foi dito/feito e o que existe.
   - **Processador:** decide encaixe, executa mudanças, e registra a ação.
5. **Separação entre determinístico e contextual é de prioridade, não de capacidade.**
   - NocoDB é a superfície preferencial para consultas factuais.
   - Lince é a superfície preferencial para busca conceitual/contextual.
   - Lince pode executar CRUD determinístico sob comando explícito.
6. **Nada de rotulagem explícita para o SER.** Campos operacionais existem internamente; a UI do SER não exibe badges/labels.
7. **Ação adequada no tempo adequado.** Nada é promovido para o canônico sem um ato explícito de esclarecimento (quando aplicável).
8. **Auditabilidade acima de conveniência.** Toda sincronização, promoção e execução gera trilha.

---

## 2. Ontologia Base (Camada Canônica)

Hierarquia ontológica do TELOS:

- **Telos** — Finalidade maior
- **Praxis** — Área contínua de atuação
- **Ergon** — Programa / sistema estruturado
- **Poiesis** — Projeto com começo, meio e fim
- **Energeia** — Ação executável (atômica)

**Nota:** Telos (no banco) é âncora estrutural singleton da instância do TELOS, não perfil do SER. É impossível existir mais de um.

Relação:

```text
Telos
 └─ Praxis
     └─ Ergon
         └─ Poiesis
             └─ Energeia
```

Energeia é entidade canônica no banco, mas pode ser visualizada como checklist textual dentro da Poiesis no Obsidian.

---

## 3. Superfícies e Componentes

### 3.1 Postgres (Canônico)

- Armazena ontologia TELOS e seus estados.
- Armazena o **espelho textual do Vault** (notas e chunks), quando desejado para recuperação semântica.
- Define integridade referencial.
- Campos denormalizados são **derivados**, nunca autorais.

### 3.2 Obsidian (Vault)

- Superfície cognitiva para texto humano (reflexão, narrativa, notas).
- Pode criar estruturas via templates/scripts.
- Não é fonte de verdade para IDs ou relações.

### 3.3 NocoDB (Determinístico, humano)

- Superfície preferencial para consultas factuais, filtros e edição administrativa.
- Ferramenta do Jardineiro (não UI do SER).

### 3.4 Vector Index (Recuperação)

- Índice **derivado** usado para busca semântica (contexto).
- Armazena vetores + metadata mínima.
- Nunca substitui o texto canônico (que vive no Vault/espelho).

### 3.5 openclaw — Lince (Memória + Processador)

O Lince opera em dois papéis complementares:

- **Memória (recuperação):**
  - prioriza busca conceitual/contextual.
  - usa o Vector Index para localizar e o Postgres/Vault para hidratar texto.

- **Processador (execução):**
  - executa ações (CRUD) sob comando explícito do SER.
  - conduz sessões de esclarecimento (clarify) e promove itens RAW para a ontologia.

**Regra:** o Lince não executa alterações silenciosas ou inferidas. Toda ação exige intenção explícita. Execução exige resolução de alvo e confirmação quando ambígua.

---

## 4. Identidade, Slug e Caminho

### 4.1 Identificadores

- `id`: UUID canônico, imutável.
- `slug`: identificador local ao escopo, sem `/`.
- `path`: caminho físico no Vault.
- `canonical_key` (opcional): identificador composto legível.

### 4.2 Unicidade por escopo

- Praxis: `(telosId, slug)`
- Ergon: `(praxisId, slug)`
- Poiesis: `(ergonId, slug)`
- Energeia: `(poiesisId, slug)`

---

## 5. Planos do Sistema

O TELOS Beta opera com três “planos” que se comunicam com contratos claros:

1. **Plano RAW (pré‑ontológico)** — captura bruta, sem cobrança.
2. **Plano Canônico (ontológico)** — Telos/Praxis/Ergon/Poiesis/Energeia.
3. **Plano de Recuperação (semântico)** — embeddings para localizar contexto.

---

## 6. RAW — Camada Pré‑Ontológica (Captura)

### 6.1 Intenção

RAW é o chão sob a árvore: tudo que cai ao redor, sem obrigação de forma.

- aceita entradas irregulares, confusas e incompletas.
- preserva o texto original.
- permite que o SER descarregue a mente com segurança.
- aguarda o momento certo de ser processado em conjunto com o Lince.

### 6.2 Entidade: RawItem

Campos mínimos sugeridos:

- `id uuid`
- `createdAt timestamptz`
- `source` (ex: `lince-chat`, `voice`, `mobile`)
- `rawText` (imutável)
- `normalizedText` (opcional; nunca substitui o raw)
- `statusInternal` (interno; ex: `captured|queued|clarified|placed|discarded`)
- `scheduledClarifyAt` (opcional)
- `placedEntityType` + `placedEntityId` (quando promovido)
- `auditRef` (opcional)

### 6.3 Regras RAW

1. **RAW não cria ontologia automaticamente.**
2. **RAW não exige triagem imediata.**
3. **RAW pode ser descartado sem culpa**, explicitamente, quando não pertence ao TELOS.
4. O valor de RAW está em **preservar**, não em classificar.

---

## 7. Esclarecimento — Sessões Clarify (Processamento Conjunto)

### 7.1 Definição

Sessão Clarify é um encontro agendado entre SER e Lince para:

- interpretar RawItems
- fazer perguntas mínimas
- decidir encaixe
- executar as mudanças necessárias
- registrar auditoria

### 7.2 Saídas possíveis por RawItem

1. **Promover** → vira entidade TELOS (Telos/Praxis/Ergon/Poiesis/Energeia)
2. **Anexar** → vira texto dentro de uma nota existente
3. **Decompor** → vira 2+ RawItems ou múltiplas Energeias
4. **Descartar** → removido do fluxo com registro (opcional)

### 7.3 Perguntas mínimas (padrão)

- Isso é contínuo (Praxis) ou finito (Poiesis)?
- Qual resultado observável?
- Existe prazo? Existe ritmo?
- Onde isso encaixa (Telos/Praxis/Ergon)?

### 7.4 Agendamento

No v0.1 o agendamento é **soft**:

- RawItem pode ter `scheduledClarifyAt`.
- O ritual pode ser diário/semana conforme o SER.

---

## 8. Plano Canônico (TELOS)

### 8.1 Modelo de dados (resumo)

- **Telos**: `id, slug, title, description`
- **Praxis**: `id, telosId, slug, title`
- **Ergon**: `id, praxisId, telosId(derivado), slug, title`
- **Poiesis**: `id, ergonId, praxisId(derivado), telosId(derivado), slug, title, statusInternal`
- **Energeia**: `id, poiesisId, slug, title, statusInternal, completedAt`

O **versionamento temporal do norte** (declarações) é **opcional** e serve apenas para auto‑conhecimento/auditoria; a ontologia e fluxos continuam funcionando mesmo sem histórico.

### 8.2 Regra de consistência

Campos derivados (`telosId`, `praxisId` em camadas inferiores) são recalculados pela API/Processador e ignoram input direto.

---

## 9. Espelho do Vault (Memória textual)

### 9.1 Intenção

O Vault é a superfície humana. O espelho no Postgres existe para:

- permitir hidratação confiável de contexto após uma busca semântica
- permitir auditoria e reindexação

### 9.2 Entidades

**Note**

- `id`
- `path unique`
- `content`
- `sha256`
- `updatedAt`

**NoteChunk**

- `id`
- `noteId`
- `chunkIndex`
- `content`
- `sha256`
- `updatedAt`

---

## 10. Plano de Recuperação (Vector Index)

- Índice por ambiente (ex: `telos-beta`).
- Namespaces por tipo (`vault-notes`, `decisions`, `raw` opcional).
- Vetores apontam para `NoteChunk.id` (ou `RawItem.id` se indexado).
- Metadata mínima: `path`, `entityType`, `telosId`, `praxisId`, `ergonId`, `poiesisId`.

O índice serve apenas para **localizar**; o texto é hidratado do Postgres/Vault.

---

## 11. Fluxos Canônicos

### 11.1 Captura (entrada livre)

1. SER envia mensagem ao Lince (dump livre).
2. Lince cria `RawItem` com `rawText`.
3. (Opcional) Lince sugere hipóteses em `normalizedText/proposedIntent` sem promover.
4. Confirma captura.

### 11.2 Clarify (sessão)

1. Lince carrega RawItems pendentes.
2. SER e Lince interpretam e decidem destino.
3. Lince executa promoção/anexo/decomposição/descarta.
4. Auditoria registra ação e referências.

### 11.3 Indexação do Vault

1. Mudança no Vault é detectada (hash/path).
2. `Note/NoteChunk` atualizados no Postgres.
3. Evento de indexação é gerado.
4. Worker atualiza Vector Index.

### 11.4 Recuperação contextual

1. Query → embedding.
2. Vector Index retorna `chunkIds`.
3. Lince hidrata chunks do Postgres.
4. Lince compõe contexto e responde.

### 11.5 Execução determinística via Lince (quando necessário)

Exemplos:

- “Marque a tarefa X como concluída.”
- “Crie uma Poiesis chamada Y dentro do Ergon Z.”

**Regra:** execução sempre sob comando explícito; reversibilidade e logs são preferíveis.

---

## 12. Frontmatter Obsidian (Contrato mínimo)

### Poiesis

```yaml
telos_id: uuid
praxis_id: uuid
ergon_id: uuid
poiesis_id: uuid
slug: rotina-inicial
```

### Energeia (visual)

```md
- [ ] Revisar plano semanal <!-- energeia_id: uuid -->
```

**Nota:** checkbox não altera canônico sem canal autorizado.

---

## 13. Conformidade TELOS — Rótulos e Estado

- `statusInternal`, `priorityInternal`, `tagsInternal` podem existir como dados.
- A UI do SER não deve exibir labels/badges.
- Representações preferenciais: ordem, agrupamento implícito, ritmo, proximidade.

---

## 14. Segurança e Permissões (v0.1)

- Sistema single-user.
- Usuários de banco com permissões mínimas.
- Execução do Lince sempre registrada.
- Recuperação contextual pode ser read-only.

---

## 15. Fora do Escopo v0.1

- Multi-tenant
- Colaboração
- Versionamento histórico profundo
- UI própria do SER

---

## 16. Encerramento

Esta especificação define o embrião do TELOS Beta:

- RAW como solo fértil (captura)
- Clarify como encontro (processo)
- Ontologia como fruto (assentamento)
- Recuperação vetorial como olfato (memória)
- Lince como memória e processador (com ação explícita)

O sistema é deliberadamente simples, para crescer com cuidado: sem ansiedade, sem rótulos, sem violência sobre o pensamento.
