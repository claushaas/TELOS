# TELOS Beta v0.1 — Patch Plan 02 (Telos Singleton Âncora + Versionamento Temporal sem Reificação)

> Este patch atualiza os documentos v0.1 para incorporar **Opção A**: manter `Telos` no Postgres como **SINGLETON âncora do sistema**, com **versionamento temporal opcional** apenas para auto‑conhecimento e auditoria histórica — **nunca** como identidade do SER e **nunca** como pré‑requisito para o funcionamento do TELOS.
>
> **Regra de precedência:** `telos_spec_v_0_1.md` continua sendo a fonte da verdade. Em caso de conflito, a SPEC prevalece.

---

## 0) O que este patch adiciona (em uma frase)

**Anti‑reificação do SER**: o TELOS jamais produz afirmações identitárias do tipo “seu telos é X, portanto você é X”; ele apenas registra escolhas/declarações **datadas** e **revisáveis**, e usa `Telos` (singleton) como âncora estrutural do grafo.

---

## 1) `telos_spec_v_0_1.md` — reforço canônico (mínimo, cirúrgico)

### 1.1 Inserir (Seção 1 — Princípios Canônicos) um novo princípio

**Adicionar após o item 2 ou 3:**

- **Anti‑reificação do SER.** O TELOS não declara “o seu telos é X”. O sistema registra apenas **declarações/compromissos datados** escolhidos pelo SER (revisáveis), e nunca os usa como atributo identitário.

### 1.2 Inserir (Seção 2 — Ontologia Base) nota de singleton

**Adicionar logo após a lista da ontologia (Telos, Praxis, …):**

- **Telos (no banco) é âncora estrutural singleton da instância do TELOS**, não perfil do SER. É impossível existir mais de um.

### 1.3 Inserir (Seção 8 — Plano Canônico) nota sobre versionamento temporal

**Adicionar após “Modelo de dados (resumo)” ou em 8.2:**

- O **versionamento temporal do norte** (declarações) é **opcional** e serve apenas para auto‑conhecimento/auditoria; a ontologia e fluxos continuam funcionando mesmo sem histórico.

---

## 2) `contratos.md` — adicionar o contrato “Anti‑Reificação” + “Telos Singleton”

### 2.1 Adicionar seção: “Anti‑Reificação do SER (Regra Fundamental)”

**Inserir após “Princípios contratuais”:**

- Proibido: frases normativas/identitárias do tipo **“seu telos é X”**, **“você é X”**, **“o sistema concluiu quem você é”**.
- Permitido: linguagem de agência e temporalidade:
  - “você **declarou** X em {data}”
  - “você **escolheu** X para este ciclo”
  - “quer **revisar** X?”
  - “hipótese: suas ações recentes **parecem** (des)alinhadas com X” (sempre como hipótese e com referências)
- O Lince deve recusar reificação mesmo quando solicitado (“defina meu telos”).

### 2.2 Adicionar seção: “Telos no Postgres: Âncora Singleton (não identidade)”

**Inserir após a seção de superfícies ou antes de Identidade:**

- Existe **exatamente 1** registro de `Telos` por instância TELOS.
- `Telos` é **âncora do grafo**, não cadastro do SER.
- A existência/edição de `Telos` não deve ser exibida como “perfil” na UI do SER.
- Qualquer histórico de norte deve ser modelado como **declarações temporais**, não como atributo permanente.

### 2.3 Adicionar pendência (se necessário) sobre enforcement

No bloco `[PENDING]` adicionar:

- **Enforcement do singleton Telos:** mecanismo (id fixo vs constraint) e regras de migração (se já existir dado).

---

## 3) `modelo-de-dados.md` — ajustar entidade Telos + adicionar entidade temporal (declarações)

### 3.1 Atualizar seção “Telos” com regra singleton

**Adicionar abaixo de `Telos` (entidade):**

- **Regra:** `Telos` é singleton (1 por instância). Implementação deve impedir criação de múltiplos registros.

### 3.2 Adicionar entidade: `TelosDeclaration` (ou equivalente)

**Inserir nova seção após “Telos” ou antes de “Praxis”:**

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

- Nunca inferir `statement`; sempre criado por comando/clarify explícito.
- O sistema funciona sem `TelosDeclaration` (histórico é opcional).

### 3.3 (Opcional) Ajustar unicidade por escopo — manter como está

Nenhuma mudança necessária, desde que `Telos` continue existindo como âncora.

---

## 4) `fluxos.md` — adicionar fluxo de “Declaração temporal do norte”

### 4.1 Inserir novo fluxo (após Clarify ou Execução determinística)

Título: “Declaração temporal do Norte (TelosDeclaration)”

Processo (alto nível):

1. SER pede explicitamente: “Quero declarar/revisar meu norte para este ciclo: …”.
2. Lince confirma intenção (especialmente se substituir o vigente).
3. Lince cria `TelosDeclaration` e registra `AuditEvent` (correlationId amarrando intenção→execução).
4. Lince responde com confirmação e referência temporal (data/ciclo), sem transformar isso em identidade.

Regra: jamais transformar declaração em atributo do SER; sempre tratar como compromisso revisável.

---

## 5) `glossario.md` — ajustar definição de Telos + incluir TelosDeclaration

### 5.1 Atualizar definição de “Telos”

Ajustar para:

- **Telos** — âncora estrutural singleton do sistema (tronco). Não é atributo identitário do SER.

### 5.2 Adicionar termo: “TelosDeclaration”

- **TelosDeclaration** — declaração temporal (datada e revisável) do norte/compromisso escolhida pelo SER; serve para auto‑conhecimento e auditoria histórica, nunca como condição de funcionamento.

### 5.3 Adicionar termo: “Anti‑Reificação”

- **Anti‑Reificação do SER** — regra que proíbe o sistema de afirmar identidades (“seu telos é X”), preservando agência e temporalidade.

---

## 6) `pendencias.md` — adicionar/atualizar pendências relacionadas

### 6.1 Adicionar item (P2 ou P1, conforme sua preferência)

[PENDING] Enforce Telos singleton + migração

- Priority: P2 (ou P1 se migrations já estiverem no foco)
- Blocks: consistência da raiz ontológica; prevenção de multi‑Telos por erro.
- Decision needed: mecanismo de enforcement e como tratar dados existentes.
- DoD: regra implementada no banco + teste/migration + contrato documentado.

### 6.2 (Opcional) Adicionar item

[PENDING] Definição de ciclos (cycle taxonomy) para TelosDeclaration

- Priority: P2
- Blocks: organização do histórico (não bloqueia funcionamento).
- Decision needed: como nomear ciclos (data, estações, Qs) sem virar rótulo.
- DoD: padrão documentado + exemplo mínimo.

---

## 7) `README.md` — pequena nota de leitura (opcional)

Adicionar 1 bullet em mapa ou ordem de implantação:

- `Telos` é âncora singleton; “norte” temporal vive em `TelosDeclaration` (opcional).

---

## 8) Checklist de commit (Patch 02)

1. `telos_spec_v_0_1.md` — princípio anti‑reificação + nota singleton + histórico opcional
2. `contratos.md` — “Anti‑Reificação” + “Telos Singleton” + pending enforcement
3. `modelo-de-dados.md` — regra singleton + `TelosDeclaration`
4. `fluxos.md` — fluxo de declaração/revisão temporal
5. `glossario.md` — termos novos
6. `pendencias.md` — pendências novas
7. `README.md` — nota (opcional)

---

## 9) Critérios de aceitação (para este patch)

- Em nenhum documento aparece ou é permitido inferir a frase “seu telos é X” como identidade.
- `Telos` é descrito explicitamente como **singleton âncora**.
- O histórico (declarações) é descrito como **opcional** e **não necessário** para funcionamento.
- O fluxo de criação/revisão de declarações exige intenção explícita e auditoria.
