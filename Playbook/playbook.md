---
tags:
  - playbook
  - indice
  - frontend
date: 2026-09-24
---

# Playbook

Indice dos guias de orientacao pro padrao generico de card frontend: qualquer card de SPA que consome um organism compartilhado (metadado de campos + entidade), independente do dominio. Cada guia responde: o que e, por que a regra existe, e quais red flags evitar.

> O conteudo original desta destrinchada veio do caso real System+Custom Fields (`spa-colab/.@convenia/.../SystemFields/`). Onde um exemplo concreto ajuda, ele aparece rotulado "Exemplo real" — mas a regra em si vale pra qualquer card, nao so esse dominio.

## Ordem de leitura recomendada

1. [[Playbook/visao-profile]] — os tokens parametrizaveis, primeiro de tudo.
2. [[Playbook/services-and-mappers]] — fetch unificado, mappers, service layer.
3. [[Playbook/store]] — shape do Vuex, naming, ordering.
4. [[Playbook/loader]] — carregamento em lote, provide/inject, options dependentes.
5. [[Playbook/container]] — o container fino, contrato de eventos.
6. [[Playbook/component-conventions]] — convencoes do lado do organism/componente (props, atributos, comentarios, imports, Storybook).
7. [[Playbook/testing]] — contrato de teste Playwright, set canonico.
8. [[Playbook/red-flags]] — checklist final antes de dar a migracao por pronta.
9. [[Playbook/cleanup]] — sweep de codigo legado, ultima etapa.

## Templates de codigo

Cada guia linka o esqueleto de codigo correspondente em [[Templates/templates]].

## Outros guias

Assunto separado do padrao card+organism acima — nivel de modulo inteiro, nao de card:

- [[Playbook/new-module]] — como criar um modulo novo (`src/<Module>/`) do zero: skeleton de pastas, wiring de aliases/store/rotas.

## Referencias cruzadas

- [[home]]
- [[Templates/templates]]
