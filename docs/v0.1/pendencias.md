# Pendências e Decisões Abertas — TELOS v0.1

> [SOURCE] Este arquivo existe para registrar o que a SPEC v0.1 (ver [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md)) **não fixa** (ainda), evitando invenção e drift.

## Modelo de dados e auditoria

- [PENDING] Definir domínio/enums para `statusInternal` (RAW e canônico) e outros campos internos mencionados na SPEC.
- [PENDING] Definir esquema mínimo de auditoria (evento, ator, timestamp, before/after, referências e `auditRef`).
- [PENDING] Definir políticas de integridade referencial (FKs e `ON DELETE/ON UPDATE`) para Telos→Praxis→Ergon→Poiesis→Energeia.

## Vault (Obsidian) → Espelho (Postgres)

- [PENDING] Definir como detectar mudanças no Vault (watcher, plugin, polling, git, etc.) e qual é o gatilho canônico.
- [PENDING] Definir critério para “espelho textual do Vault (quando desejado)” e como habilitar/desabilitar por ambiente.
- [PENDING] Definir estratégia de chunking (`NoteChunk`) e quando re-chunkar (mudança parcial vs total).

## Vector Index (recuperação)

- [PENDING] Decidir provider e topologia: Vectorize vs pgvector (ou híbrido), e implicações de ambiente/namespace.
- [PENDING] Definir se RAW entra no índice semântico (SPEC diz “raw opcional”) e quais salvaguardas de privacidade/auditoria.
- [PENDING] Definir metadados mínimos obrigatórios por namespace (além dos citados na SPEC, se necessário).

## Lince (openclaw): Memória + Processador

- [PENDING] Definir protocolo de confirmação para “execução sob comando explícito” (ex.: confirmação dupla para CRUD).
- [PENDING] Definir como sessões Clarify são selecionadas (prioridade por tempo, por energia, por área, etc.) sem introduzir “rótulos para o SER”.

## NocoDB (determinístico)

- [PENDING] Definir quais tabelas/visões são expostas no NocoDB (e quais são estritamente internas, ex.: auditoria).

## Segurança (v0.1)

- [PENDING] Definir usuários/papéis do banco e permissões mínimas por superfície (SER, Lince, workers de indexação).
- [PENDING] Definir quais operações de “recuperação contextual” são read-only e em que condições.
