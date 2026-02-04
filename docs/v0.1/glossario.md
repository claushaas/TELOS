# Glossário — TELOS v0.1

> [SOURCE] Derivado de [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md). Em caso de conflito, a SPEC prevalece.

## Ontologia (camada canônica)

- **Telos** — âncora estrutural singleton do sistema (tronco). Não é atributo identitário do SER.
- **Praxis** — área contínua de atuação.
- **Ergon** — programa / sistema estruturado.
- **Poiesis** — projeto com começo, meio e fim.
- **Energeia** — ação executável (atômica).

## Temporalidade e anti-reificação

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 5). Em caso de conflito, a SPEC prevalece.

- **TelosDeclaration** — declaração temporal (datada e revisável) do norte/compromisso escolhida pelo SER; serve para auto‑conhecimento e auditoria histórica, nunca como condição de funcionamento.
- **Anti‑Reificação do SER** — regra que proíbe o sistema de afirmar identidades (“seu telos é X”), preservando agência e temporalidade.

## Planos (camadas do sistema)

- **Plano RAW (pré‑ontológico)** — camada de captura bruta; preserva texto sem competir com a ontologia.
- **Plano Canônico (ontológico)** — camada de entidades Telos/Praxis/Ergon/Poiesis/Energeia no Postgres.
- **Plano de Recuperação (semântico)** — camada derivada (vetores) usada para busca contextual.

## Componentes / superfícies

- **Postgres (canônico)** — fonte única de verdade para ontologia, relações e estados canônicos.
- **Obsidian (Vault)** — superfície cognitiva humana para texto; não é fonte de verdade para IDs/relações.
- **NocoDB** — superfície preferencial para consultas factuais e edição administrativa (ferramenta do Jardineiro).
- **Vector Index** — índice derivado para localizar contexto semântico (ex.: Vectorize/pgvector).
- **openclaw (Lince)** — agente que opera como Memória (recuperação) e Processador (execução) sob intenção explícita.

## Conceitos operacionais

- **RAW** — “solo” pré‑ontológico; aceitação de entrada irregular; preservação acima de classificação.
- **RawItem** — entidade canônica de captura bruta (`rawText` imutável) no Plano RAW.
- **Sessão Clarify** — encontro agendado entre SER e Lince para interpretar RawItems, decidir encaixe e executar mudanças com auditoria.
- **Memória** — papel do Lince focado em localizar/hidratar contexto (busca conceitual/contextual).
- **Processador** — papel do Lince focado em executar mudanças (CRUD) e registrar ações (auditoria).
- **Determinístico** — consultas/edições factuais e estruturadas; NocoDB é a superfície preferencial.
- **Contextual** — busca semântica/conceitual; Lince + Vector Index é a superfície preferencial.

## Identidade e endereçamento

- **`id`** — UUID canônico e imutável.
- **`slug`** — identificador local ao escopo, sem `/`.
- **`path`** — caminho físico no Vault.
- **`canonical_key` (opcional)** — identificador composto legível.

## Espelho do Vault

- **Note** — espelho (no Postgres) de uma nota do Vault (`path`, `content`, `sha256`, etc.).
- **NoteChunk** — fragmento (chunk) de Note para indexação e hidratação contextual.

## Auditoria

- **Auditabilidade** — prioridade de registrar sincronizações, promoções e execuções (trilha de auditoria).
- **`auditRef` (opcional)** — referência para amarrar ações e registros de auditoria.
- **AuditEvent** — registro atômico de uma ação do sistema (ator + motivo + before/after + correlação).
- **Undo / Tombstone** — reversibilidade mínima por marcação de deleção (ex.: `deletedAt`) e/ou janela de desfazer.
- **Referência humana (`human_ref`)** — referência textual do SER a um alvo (“tarefa X”), a ser resolvida com confirmação quando ambígua.
