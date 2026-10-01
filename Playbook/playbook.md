---
tags:
  - playbook
  - indice
  - frontend
date: 2026-09-24
---

# Playbook

Índice dos guias de orientação pro padrão genérico de card frontend: qualquer card de SPA que consome um organism compartilhado (metadado de campos + entidade), independente do domínio. Cada guia responde: o que é, por que a regra existe, e quais red flags evitar.

> O conteúdo original desta destrinchada veio do caso real System+Custom Fields (`spa-colab/.@convenia/.../SystemFields/`). Onde um exemplo concreto ajuda, ele aparece rotulado "Exemplo real" — mas a regra em si vale pra qualquer card, não só esse domínio.

## Ordem de leitura recomendada

1. [[Playbook/visao-profile]] — os tokens parametrizáveis, primeiro de tudo.
2. [[Playbook/services]] — fetch unificado de metadado, service layer, contrato `[ err, data ]`.
3. [[Playbook/mappers]] — mappers de leitura/escrita em `content/mappers`.
4. [[Playbook/store]] — shape do Vuex, naming, ordering.
5. [[Playbook/loader]] — carregamento em lote, provide/inject, options dependentes.
6. [[Playbook/container]] — o container fino, contrato de eventos.
7. [[Playbook/component-conventions]] — convenções do lado do organism/componente (props, atributos, comentários, imports, Storybook).
8. [[Playbook/testing]] — contrato de teste Playwright, set canônico.
9. [[Playbook/red-flags]] — checklist final antes de dar a migração por pronta.
10. [[Playbook/cleanup]] — sweep de código legado, última etapa.

## Templates de código

Cada guia linka o boilerplate de código correspondente em [[Templates/templates]].

## Outros guias

Assunto separado do padrão card+organism acima — nível de módulo inteiro, não de card:

- [[Playbook/new-module]] — como criar um módulo novo (`src/<Module>/`) do zero: skeleton de pastas, wiring de aliases/store/rotas.

## Referências cruzadas

- [[home]]
- [[Templates/templates]]
