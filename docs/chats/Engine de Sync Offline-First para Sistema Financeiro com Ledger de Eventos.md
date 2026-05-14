---
title: "Engine de Sync Offline-First para Sistema Financeiro com Ledger de Eventos"
project: STOA
type: conversation-distillation
date: 2026-02-25
source: chatgpt
tags: [stoa, arquitetura, filosofia, decisões]
---
# 1. Panorama Geral

A conversa teve como propósito central definir a arquitetura de uma engine de sincronização para um sistema financeiro offline-first, com foco em:

- Dados financeiros armazenados localmente (SQLite).
- Sincronização com backend em Postgres via HTTP.
- Agente sempre online, mas operando apenas por geração de eventos.
- Arquitetura baseada em event log (ledger) como fonte de verdade.
- Capacidade de plugar múltiplas fontes de input e output no futuro.
- Determinismo estrutural: o estado demonstrado deve ser idêntico independentemente da superfície (frontend).

A discussão evoluiu de escolhas técnicas (SQLite vs MMKV) até a formalização de uma SPEC completa da engine.

---

# 2. Assuntos Abordados

## 2.1 SQLite vs MMKV como armazenamento local

- **Definição do tema:** escolha da tecnologia de persistência local para dados financeiros offline-first.
- **Problema tratado:** qual banco local combina melhor com Postgres e com um modelo sincronizável.
- **Contexto:** necessidade de queries complexas, projeções, integridade e sync robusto.
- **Implicações:**
  - SQLite é relacional, transacional (ACID) e adequado para ledger + projeções.
  - MMKV é adequado apenas como KV auxiliar (tokens, flags, estado de sync).
  
[Decisão]  
SQLite será o armazenamento local principal.  
MMKV/AsyncStorage será usado apenas como KV auxiliar.

---

## 2.2 Arquitetura SQLite + Postgres com Sync por Eventos

- **Definição do tema:** desenho de alto nível da integração entre SQLite local e Postgres remoto.
- **Problema tratado:** como garantir sincronização robusta, idempotente e auditável.
- **Contexto:** múltiplos dispositivos, reenvio, quedas de rede, duplicação.
- **Implicações:**
  - Sync baseado em push/pull de eventos.
  - Idempotência via `event_id`.
  - Ordenação via `device_id + device_seq` e `server_seq`.
  - Checkpoints e cursores por dispositivo e por usuário.

[Decisão]  
A sincronização será baseada exclusivamente em eventos append-only.

---

## 2.3 Evento vs Transação

- **Definição do tema:** distinção conceitual entre evento e transação.
- **Problema tratado:** evitar confundir estado final com fato imutável.
- **Contexto:** modelagem de sistema financeiro sincronizável.

### Formulação central:
> "Transação não é um fato. Transação é uma interpretação atual do histórico de fatos."

- **Implicações:**
  - Transação é projeção.
  - Evento é fato imutável.
  - Sync trabalha com eventos, não estados finais.

[Decisão]  
Modelo baseado em ledger de eventos com projeções derivadas.

---

## 2.4 Retenção de dados no dispositivo (Hot Window)

- **Definição do tema:** limitação do range de dados no device.
- **Problema tratado:** manter performance e leveza sem comprometer consistência.
- **Contexto:** finanças exigem saldo correto mesmo com retenção.

Estratégia definida:

- Hot window fixo (ex.: 90 dias).
- Âncoras de saldo (`AccountBalanceAnchorSet`).
- Hidratação sob demanda (`/sync/hydrate`).
- Server mantém histórico completo.

[Decisão]  
Device funciona como cache inteligente com âncoras, não como repositório histórico completo.

---

## 2.5 Engine plugável com múltiplas fontes de input/output

- **Definição do tema:** arquitetura capaz de integrar múltiplos inputs (manual, agente, importações).
- **Problema tratado:** garantir consistência independente da origem.
- **Contexto:** desejo de futura expansão.

[Decisão]  
Toda entrada vira evento no mesmo formato canônico (`EventEnvelope`).  
Toda saída lê projeções derivadas.

---

## 2.6 SPEC MVP (Completo, mas com 1 adapter por tipo)

Escopo definido:

✅ Core:
- Event model
- Sync engine push/pull
- Cursor/checkpoint
- Retention policy fixa
- Conflict strategy (OCC)
- Projection engine
- Idempotência
- Observabilidade

✅ Adapter local:
- SQLite

✅ Adapter remoto:
- HTTP API para Postgres

✅ KV auxiliar:
- MMKV/AsyncStorage

[Decisão]  
MVP não significa simplificação estrutural. Significa apenas um adapter por categoria.

---

# 3. Conceitos e Modelos Mentais

## 3.1 Ledger como Fonte de Verdade

**Definição operacional:**  
Sequência append-only de eventos imutáveis.

**Metáfora:**  
"Ledger = sequência de fatos. Transações = visão atual desses fatos."

**Relação com sync:**  
Sincroniza-se fatos, não estados.

---

## 3.2 Projeção

**Definição operacional:**  
Estado atual derivado da aplicação ordenada de eventos.

**Exemplo conceitual:**

```
TransactionCreated
TransactionUpdated
TransactionCategorized
```

→ Estado final projetado.

**Implicação:**  
Projeções devem ser rebuildáveis.

---

## 3.3 Idempotência

**Definição operacional:**  
Aplicar o mesmo evento duas vezes não altera o estado.

**Implementação:**
- `event_id` único.
- `UNIQUE(user_id, device_id, device_seq)`.

---

## 3.4 Convergência

**Definição operacional:**  
Dispositivos autorizados convergem para o mesmo estado após sync.

**Dependências:**
- Ordenação determinística.
- OCC para conflitos reais.
- Projeções puramente determinísticas.

---

## 3.5 Hot Window + Âncoras

**Definição operacional:**  
Retenção parcial de eventos no device com manutenção de saldos via âncoras.

**Relação estrutural:**  
Permite performance sem perder consistência contábil.

---

# 4. Decisões Tomadas

## 4.1 SQLite como DB local principal
- **Racional:** integridade, queries ricas, compatibilidade conceitual com Postgres.
- **Trade-off:** maior complexidade que KV simples.
- **Consequência:** suporte real a ledger + projeções.

---

## 4.2 Event Sourcing como base estrutural
- **Racional:** auditabilidade, sync robusto, plugabilidade.
- **Trade-off:** maior complexidade inicial.
- **Consequência:** sistema evolutivo e determinístico.

---

## 4.3 Sync baseado em Push/Pull + Cursores
- **Racional:** controle explícito, tolerância a falhas.
- **Trade-off:** necessidade de controle rigoroso de ACK e ordenação.
- **Consequência:** tolerância a duplicação e quedas de rede.

---

## 4.4 OCC (Optimistic Concurrency Control)
- **Racional:** evitar merge silencioso em finanças.
- **Trade-off:** fluxo de conflito explícito.
- **Consequência:** conflitos são eventos, não efeitos colaterais.

---

# 5. Hipóteses e Direções em Aberto

- [Hipótese] Evolução futura para múltiplos adapters locais (IndexedDB, SQLite WASM).
- [Hipótese] Possível uso de hash chain (`prev_hash`) para integridade criptográfica.
- [Hipótese] Evolução para double-entry contábil no futuro.
- [Hipótese] Compaction avançada local além de hot window simples.

---

# 6. Tensões e Dilemas Estruturais

## 6.1 Simplicidade vs Rigor Contábil
Tensão entre:
- modelo simples de transações
- robustez completa de event sourcing

Decisão inclinada ao rigor estrutural.

---

## 6.2 Retenção Local vs Reconstrução Total
Manter todos eventos localmente vs usar âncoras + hidratação.

Escolha: retenção parcial com âncoras.

---

## 6.3 Merge automático vs conflito explícito
Escolha clara por conflito explícito.

---

# 7. Frases de Impacto e Formulações Nucleares

### 7.1
**Versão exata:**  
"Transação não é um fato. Transação é uma interpretação atual do histórico de fatos."

**Significado:**  
Separação rigorosa entre fato e estado derivado.

---

### 7.2
"Ledger é a verdade. Projeções são visões."

**Significado:**  
Hierarquia epistemológica do sistema.

---

### 7.3
"Você sincroniza eventos, não estados finais."

**Significado:**  
Princípio central da engine.

---

# 8. Mudanças de Direção

## Modelo anterior implícito
Transação poderia ser tratada como unidade primária sincronizável.

## Novo modelo
Eventos são a unidade primária.  
Transação é projeção.

## Razão
Necessidade de:
- auditabilidade
- idempotência
- convergência multi-device
- retenção inteligente

---

# 9. Implicações para o Projeto STOA

- Criação de biblioteca `@finance-sync/core`.
- Separação clara entre:
  - domínio (ledger)
  - persistência
  - sync
  - UI
- Sistema portável entre frontends.
- Arquitetura alinhada com princípios de:
  - rastreabilidade
  - estabilidade estrutural
  - não-magia
  - determinismo

---

# 10. Síntese Estrutural Final

A conversa consolidou uma arquitetura onde:

- O sistema financeiro é um **ledger append-only**.
- O estado visível é uma **projeção determinística**.
- A sincronização é feita por **troca idempotente de eventos com cursores explícitos**.
- O dispositivo funciona como **cache inteligente com hot window e âncoras**.
- Conflitos são tratados como eventos explícitos.
- A engine é plugável e agnóstica de frontend.
- SQLite é o armazenamento local canônico.
- Postgres é o armazenamento remoto append-only.
- KV auxiliar é separado e não contém dados financeiros.

A estrutura resultante é consistente, auditável, evolutiva e preparada para múltiplas fontes de entrada e saída, mantendo invariância de estado em qualquer superfície.