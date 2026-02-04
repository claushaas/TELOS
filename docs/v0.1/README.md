# TELOS — Documentação v0.1

> **Status:** Beta operacional (embrião)
>
> **Fonte da verdade (v0.1):** [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md)

## Como ler esta pasta

- Se você vai **implementar** ou **avaliar consistência**, comece pela SPEC.
- Os demais arquivos aqui são **derivados** da SPEC e existem para facilitar navegação e execução; em caso de conflito, a SPEC prevalece.

## Mapa de documentos (grafo v0.1)

### SPEC (canônica)

- [`telos_spec_v_0_1.md`](./telos_spec_v_0_1.md) — Especificação técnica v0.1 (contratos e invariantes).

### Reference (derivadas)

- [`glossario.md`](./glossario.md) — Vocabulário canônico usado na spec.
- [`modelo-de-dados.md`](./modelo-de-dados.md) — Entidades/campos/relações e regras de unicidade/derivação.
- [`contratos.md`](./contratos.md) — Contratos entre superfícies (Postgres/Vault/NocoDB/Vector/Lince) e invariantes de execução.
- [`fluxos.md`](./fluxos.md) — Fluxos canônicos (captura, clarify, indexação, recuperação, execução determinística).

### Backlog de decisões

- [`pendencias.md`](./pendencias.md) — Questões abertas e decisões necessárias para implementação sem inventar regra.

## Contexto fora de v0.1 (não-canônico para esta pasta)

Estes documentos ajudam a entender “por que” e princípios, mas não substituem a SPEC v0.1:

- [`README.md`](../../README.md) (raiz) — visão geral do TELOS.
- [`WHY_TELOS.md`](../../WHY_TELOS.md) — fundamentação filosófica e metafísica.
- [`anti-TELOS.md`](../../anti-TELOS.md) — non-goals e recusas fundamentais.
- [`manifesto-anti-marketing.md`](../../manifesto-anti-marketing.md) — manifesto (comunicação/princípios).
- [`docs/architecture/principios-arquiteturais-da-infraestrutura.md`](../architecture/principios-arquiteturais-da-infraestrutura.md) — princípios arquiteturais (infra).
- [`docs/architecture/pincipios-arquiteturais-da-interface-cognitiva.md`](../architecture/pincipios-arquiteturais-da-interface-cognitiva.md) — princípios arquiteturais (interface cognitiva).

## Convenções de marcação

- **[SOURCE]** — referência explícita ao trecho/arquivo que fundamenta a afirmação.
- **[PENDING]** — falta decisão ou dado; não inventar.
- **[ASSUMPTION]** — suposição de baixo risco, explicitada; revisar assim que houver definição.
- **[RISK]** — risco conhecido por falta de validação/detalhe.
