---
title: "Pragmata, Hexis e Prattein — Nomenclatura Aristotélica para Monorepo e CLI"
project: STOA
type: conversation-distillation
date: 2026-02-25
source: chatgpt
tags: [stoa, arquitetura, filosofia, decisões]
---

# 1. Panorama Geral

A conversa teve como propósito definir nomenclaturas aristotélicas rigorosas para:

- Um monorepo que agrega skills, prompts, instruções e plugins relacionados a IA.
- Um folder destinado a arquivos de identidade do agente (ex.: `agents.md`, `soul.md`, `identity.md`).
- O verbo adequado para representar o ato de executar o sistema via CLI.
- A reformulação aristotélica de um comando no formato: `"use pragmata X para Y"`.

O eixo central foi alinhar arquitetura técnica com ontologia aristotélica, evitando termos vagos como “stuff” e substituindo-os por categorias semanticamente precisas.

---

# 2. Assuntos Abordados

## 2.1 Nome aristotélico para “stuff” (monorepo)

- **Tema:** Nomear um monorepo que agrega múltiplos elementos operacionais.
- **Problema:** “claus-ai-stuff” é semanticamente fraco.
- **Contexto:** Desejo de seguir nomenclatura aristotélica coerente com TELOS/STOA.
- **Implicações:** O nome deve carregar ontologia implícita e não soar genérico.

Foram apresentados termos gregos possíveis:

- _Prágmata_ (πράγματα)
- _Erga_ (ἔργα)
- _Heurémata_ (εὑρήματα)
- _Organon_ (ὄργανον)
- _Hypokeímena_ (ὑποκείμενα)

[Decisão] O usuário escolheu imediatamente:

> “nem li o resto, já fiquei com pragmata”

Conclusão: o monorepo passa a se chamar conceitualmente **`claus-pragmata`**.

---

## 2.2 Nome para folder de identidade do agente

- **Tema:** Nome para diretório que contém arquivos de identidade.
- **Problema:** Como nomear algo que agrega `agents.md`, `soul.md`, `identity.md`?
- **Definição ontológica estabelecida:**
  - Não é configuração.
  - Não é prompt.
  - É “aquilo que define o que o agente é enquanto agente”.
  - É “o princípio de identidade que permanece estável mesmo quando o comportamento varia”.

Termos avaliados:

- _Hexis_ (ἕξις)
- _Ethos_ (ἦθος)
- _Ousia_ (οὐσία)
- _Psyche_ (ψυχή)
- _Arche_ (ἀρχή)

[Inferência fundamentada] O termo mais adequado foi **hexis**, definido como “disposição estável do ser”.

Exemplo estrutural sugerido:

```txt
hexis/
  agents.md
  identity.md
  soul.md
```

---

## **2.3 Verbo para o uso de pragmata**

- **Tema:** Quando pragmata está em uso via CLI, qual é o verbo adequado?
- **Problema:** “use” é semanticamente fraco.
- **Contexto:** Diferenciar produzir, fabricar e agir.

Foi estabelecido:

- _Prágma_ (πρᾶγμα) → coisa enquanto assunto de ação
- _Práttein_ (πράττειν) → agir, executar ação deliberada

Conclusão operacional:

O verbo correto para o uso de pragmata é:

> **πράττειν — práttein**

Não é:
- produzir
- fabricar
- gerar

É agir deliberadamente.

---

## **2.4 Reformulação aristotélica do comando CLI**

Comando original:

> “use pragmata X para Y”

Crítica semântica:
- “use” é fraco.
    
- “para Y” é finalidade rasa.

Forma aristotélica proposta:

> **πράττειν τὰ πράγματα Χ πρὸς Υ**

Tradução:

> “agir com os prágmata X em vista de Y”

Estrutura semântica:
- πράττειν → agir
    
- τὰ πράγματα Χ → os pragmata X
    
- πρὸς Υ → orientado a Y (finalidade teleológica)

Versões CLI sugeridas:

```
prattein pragmata:X pros:Y
```

ou

```
prattein X pros Y
```

---

# **3. Conceitos e Modelos Mentais**

## **3.1 Prágmata (πράγματα)**

**Definição operacional:**

Coisas enquanto assuntos de ação; coisas-em-uso.

**Metáfora associada:**

“Não é objeto passivo; é coisa enquanto problema, tarefa, uso.”

**Relação com outros conceitos:**
- Relaciona-se diretamente a _práttein_.
- Não é acervo estático; é campo de ação.

**Evolução:**

Escolhido como nome do monorepo principal.

---

## **3.2 Hexis (ἕξις)**

**Definição operacional:**

Disposição estável que molda o agir.

**Metáfora associada:**

Aquilo que o agente “tem” e que orienta sua ação.

**Relação com outros conceitos:**
- Sustenta o agir.
- Não cristaliza identidade como _ousia_.

**Evolução:**

Proposto como nome para folder de identidade.

---

## **3.3 Práttein (πράττειν)**

**Definição operacional:**

Agir deliberadamente em um contexto.

**Metáfora associada:**

Ação orientada, não fabricação.

**Relação com outros conceitos:**
- Opera sobre _prágmata_.
- Diferencia-se de _poiein_ (produzir).
---

## **3.4 Poiesis vs Praxis (implícito)**

[Inferência fundamentada]

O diálogo estabelece distinção implícita:

- **Poiesis** → produção de artefato.
- **Praxis / Práttein** → ação deliberada com finalidade.

O sistema CLI é enquadrado como praxis, não fábrica.

---

# **4. Decisões Tomadas**

  

## **4.1 Nome do monorepo**

[Decisão]

- Nome: claus-pragmata

**Racional explícito:**
- Evitar “stuff”.
- Alinhar com ontologia aristotélica.
- Nome que “já nasce operando”.

**Trade-offs:**
- Menos imediato para leigos.
- Mais consistente conceitualmente.

---

## **4.2 Nome do folder de identidade**

[Inferência fundamentada]

- Nome recomendado: hexis/

**Racional:**
- Representa disposição estável.
- Evita fixação ontológica excessiva como _ousia_.

---

## **4.3 Verbo para execução CLI**

[Conclusão conceitual]
- Verbo adequado: _práttein_

**Racional:**

- CLI representa ação deliberada.
- Output é efeito colateral do agir.

---

# **5. Hipóteses e Direções em Aberto**

- [Hipótese] Estrutura tripla futura:

```
hexis/     # quem o agente é
praxis/    # como ele age
pragmata/  # com o que ele lida
```

- [Hipótese] Evolução de CLI para estrutura totalmente grega (prattein X pros Y).

---

# **6. Tensões e Dilemas Estruturais**

## **6.1 Identidade fixa vs identidade operativa**

Tensão entre:

- _Ousia_ (substância fixa)
- _Hexis_ (disposição dinâmica)

A escolha por hexis evita cristalização do SER.

---

## **6.2 Produção vs Ação**

Tensão entre:
- CLI como fábrica (poiesis)
- CLI como ação deliberada (praxis)

A conversa resolve a favor da segunda.

---

# **7. Frases de Impacto e Formulações Nucleares**

## **7.1 “Prágmata — coisas-em-uso”**

**Versão exata:**

> “coisas-em-uso”

**Significado:**

Não são objetos passivos; são elementos enquanto operam.

---

## **7.2 “Nome que já nasce operando”**

**Significado:**

O nome carrega ação implícita.

---

## **7.3 “Você não está rodando um compilador. Está agindo em um contexto.”**

**Significado:**

Reenquadramento ontológico do CLI.

---

## **7.4 “O resto — arquivos, prompts, outputs — são apenas efeitos do práttein.”**

**Significado:**

Output não é o centro; ação é o centro.

---

# **8. Mudanças de Direção**

## **8.1 De “stuff” para ontologia**

**Modelo anterior:**

Nome genérico (“claus-ai-stuff”).

**Novo modelo:**

Nome ontológico: claus-pragmata.

**Razão:**

Precisão conceitual e alinhamento filosófico.

---

# **9. Implicações para o Projeto STOA**

- Consolidação de nomenclatura aristotélica como padrão estrutural.
- Diferenciação clara entre:
    - identidade (hexis)
    - campo de ação (pragmata)
    - ato (prattein)
- Possibilidade de CLI semanticamente coerente com a ontologia do sistema.
- Fortalecimento da integração entre filosofia e arquitetura técnica.

---

# **10. Síntese Estrutural Final**

A conversa estabelece um micro-sistema aristotélico aplicado à arquitetura de software:

- **Prágmata** → o campo das coisas enquanto operam.
- **Hexis** → a disposição estável que orienta o agente.
- **Práttein** → o ato deliberado de agir via sistema.

O CLI deixa de ser fábrica de artefatos e passa a ser ato teleológico.
O output deixa de ser centro ontológico e torna-se consequência do agir.

O resultado é uma estrutura conceitualmente coesa onde nome, ação e finalidade estão alinhados sob uma mesma gramática filosófica.