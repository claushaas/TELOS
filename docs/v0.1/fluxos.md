# Fluxos Canônicos — TELOS v0.1

> [SOURCE] Derivado de [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) (seção 11). Em caso de conflito, a SPEC prevalece.

## 1) Captura (entrada livre)

[SOURCE] SPEC v0.1 — “Captura (entrada livre)”.

### **Entrada**

- SER envia mensagem ao Lince (dump livre).

### **Processo**

- Lince cria `RawItem` com `rawText`.
- (Opcional) Lince sugere hipóteses em `normalizedText` sem promover para ontologia.

### **Saída**

- Confirmação de captura (RAW preservado).

## 2) Clarify (sessão)

[SOURCE] SPEC v0.1 — “Clarify (sessão)” + seção 7.

### **Entrada**

- RawItems pendentes (e, opcionalmente, `scheduledClarifyAt`).

### **Processo**

1. Lince carrega RawItems pendentes.
2. SER e Lince interpretam e decidem destino.
3. Lince executa **promoção/anexo/decomposição/descartar** sob intenção explícita.

### **Saída (por RawItem)**

- Promover → vira entidade TELOS (Telos/Praxis/Ergon/Poiesis/Energeia)
- Anexar → vira texto dentro de nota existente
- Decompor → vira 2+ RawItems ou múltiplas Energeias
- Descartar → removido do fluxo (com registro opcional)

### **Perguntas mínimas (padrão)**

- Isso é contínuo (Praxis) ou finito (Poiesis)?
- Qual resultado observável?
- Existe prazo? Existe ritmo?
- Onde isso encaixa (Telos/Praxis/Ergon)?

## 3) Indexação do Vault

[SOURCE] SPEC v0.1 — “Indexação do Vault”.

### **Processo**

1. Mudança no Vault é detectada (hash/path).
2. `Note/NoteChunk` são atualizados no Postgres.
3. Evento de indexação é gerado.
4. Worker atualiza o Vector Index.

## 4) Recuperação contextual

[SOURCE] SPEC v0.1 — “Recuperação contextual”.

### **Processo**

1. Query → embedding.
2. Vector Index retorna `chunkIds`.
3. Lince hidrata chunks do Postgres.
4. Lince compõe contexto e responde.

## 5) Execução determinística via Lince (quando necessário)

[SOURCE] SPEC v0.1 — “Execução determinística via Lince”.

### **Exemplos**

- “Marque a tarefa X como concluída.”
- “Crie uma Poiesis chamada Y dentro do Ergon Z.”

### **Regra**

- Execução sempre sob comando explícito; reversibilidade e logs são preferíveis.

## 6) Declaração temporal do Norte (TelosDeclaration)

[SOURCE] Derivado de [`TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md`](./TELOS-beta-v0.1-patch-plan-02-telos-singleton-temporal.md) (seção 4.1) + [`contratos.md`](./contratos.md).

### **Processo (alto nível)**

1. SER pede explicitamente: “Quero declarar/revisar meu norte para este ciclo: …”.
2. Lince confirma intenção (especialmente se substituir a vigente).
3. Lince cria `TelosDeclaration` e registra `AuditEvent` (`correlationId` amarrando intenção → execução).
4. Lince responde com confirmação e referência temporal (data/ciclo), sem transformar isso em identidade.

### **Regra**

- Jamais transformar declaração em atributo identitário do SER; tratar sempre como compromisso revisável.

## 7) Resolução de alvo + confirmação

### **Objetivo**

Executar comandos como “marque a tarefa X como concluída” sem inventar alvo e com auditoria.

### **Processo (alto nível)**

1. SER envia comando com alvo por `id`/`path`/`canonical_key` ou por referência humana (`human_ref`).
2. Lince tenta resolução determinística (quando possível).
3. Se houver múltiplos candidatos ou ambiguidade, Lince solicita confirmação explícita do SER antes de executar.
4. Lince registra auditoria (antes/depois) com `correlationId` para amarrar intenção → execução.
5. Lince executa a mudança e responde com o resultado (e caminho de reversão, quando aplicável).

## 8) Conflito de sincronização (Vault ↔ Postgres)

### **Cenários**

- **Vault mudou e o espelho está desligado:** registrar evento auditável de mudança (ex.: `path` + hash) e **deferir** espelhamento/chunking/indexação até o espelho ser habilitado.
- **Nota mudou mas chunking não está pronto:** registrar evento e **deferir/retry** a criação de `NoteChunk` e reindexação.
- **Marcação de reindexação:** produzir um evento/outbox para reindexação assíncrona (evitar “reindex silenciosa” sem trilha).

## [PENDING] Detalhes que precisam ser definidos para operacionalizar

- Como identificar “intenção explícita” de forma não ambígua (frase-padrão, confirmação dupla, modo de execução, etc.).
- Definição de ciclos (cycle taxonomy) para `TelosDeclaration` (opcional).
- Resolução de referência (alvos humanos / “tarefa X”) e regra de confirmação sob ambiguidade.
- Conflito checkbox do Obsidian ↔ status canônico no Postgres (Energeia).
- Reversibilidade mínima/undo (tombstone, janela de undo e auditoria correlacionada).
- Modelo mínimo de auditoria por fluxo (quais eventos são obrigatórios).
- Estratégia de chunking para `NoteChunk` (tamanho, delimitadores, hashing).
