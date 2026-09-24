---
tags:
  - playbook
  - indice
  - frontend
date: 2026-09-24
---

# Playbook

Indice dos guias de orientacao pro padrao de card frontend (SPA consome um organism de `@convenia/components`). Cada guia responde: o que e, por que a regra existe, e quais red flags evitar.

## Ordem de leitura recomendada

1. [[Playbook/visao-profile]] — os tokens parametrizaveis, primeiro de tudo.
2. [[Playbook/services-and-mappers]] — fetch unificado, mappers, service layer.
3. [[Playbook/store]] — shape do Vuex, naming, ordering.
4. [[Playbook/loader]] — carregamento em lote, provide/inject, options dependentes.
5. [[Playbook/container]] — o container fino, contrato de eventos.
6. [[Playbook/testing]] — contrato de teste Playwright, set canonico.
7. [[Playbook/red-flags]] — checklist final antes de dar a migracao por pronta.
8. [[Playbook/cleanup]] — sweep de codigo legado, ultima etapa.

## Templates de codigo

Cada guia linka o esqueleto de codigo correspondente em [[Templates/templates]].

## Referencias cruzadas

- [[home]]
- [[Templates/templates]]
