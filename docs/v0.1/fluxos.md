# Fluxos Canônicos — TELOS v0.1

> [SOURCE] Derivado de [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) (seção 11). Em caso de conflito, a SPEC prevalece.

## 1) Captura (entrada livre)

[SOURCE] SPEC v0.1 — “Captura (entrada livre)”.

**Entrada**

- SER envia mensagem ao Lince (dump livre).

**Processo**

- Lince cria `RawItem` com `rawText`.
- (Opcional) Lince sugere hipóteses em `normalizedText` sem promover para ontologia.

**Saída**

- Confirmação de captura (RAW preservado).

## 2) Clarify (sessão)

[SOURCE] SPEC v0.1 — “Clarify (sessão)” + seção 7.

**Entrada**

- RawItems pendentes (e, opcionalmente, `scheduledClarifyAt`).

**Processo**

1. Lince carrega RawItems pendentes.
2. SER e Lince interpretam e decidem destino.
3. Lince executa **promoção/anexo/decomposição/descartar** sob intenção explícita.

**Saída (por RawItem)**

- Promover → vira entidade TELOS (Telos/Praxis/Ergon/Poiesis/Energeia)
- Anexar → vira texto dentro de nota existente
- Decompor → vira 2+ RawItems ou múltiplas Energeias
- Descartar → removido do fluxo (com registro opcional)

**Perguntas mínimas (padrão)**

- Isso é contínuo (Praxis) ou finito (Poiesis)?
- Qual resultado observável?
- Existe prazo? Existe ritmo?
- Onde isso encaixa (Telos/Praxis/Ergon)?

## 3) Indexação do Vault

[SOURCE] SPEC v0.1 — “Indexação do Vault”.

**Processo**

1. Mudança no Vault é detectada (hash/path).
2. `Note/NoteChunk` são atualizados no Postgres.
3. Evento de indexação é gerado.
4. Worker atualiza o Vector Index.

## 4) Recuperação contextual

[SOURCE] SPEC v0.1 — “Recuperação contextual”.

**Processo**

1. Query → embedding.
2. Vector Index retorna `chunkIds`.
3. Lince hidrata chunks do Postgres.
4. Lince compõe contexto e responde.

## 5) Execução determinística via Lince (quando necessário)

[SOURCE] SPEC v0.1 — “Execução determinística via Lince”.

**Exemplos**

- “Marque a tarefa X como concluída.”
- “Crie uma Poiesis chamada Y dentro do Ergon Z.”

**Regra**

- Execução sempre sob comando explícito; reversibilidade e logs são preferíveis.

## [PENDING] Detalhes que precisam ser definidos para operacionalizar

- Como identificar “intenção explícita” de forma não ambígua (frase-padrão, confirmação dupla, modo de execução, etc.).
- Modelo mínimo de auditoria por fluxo (quais eventos são obrigatórios).
- Estratégia de chunking para `NoteChunk` (tamanho, delimitadores, hashing).
