---
tags:
  - templates
  - indice
  - frontend
date: 2026-10-01
---

# Templates

Boilerplates de código prontos pra copiar, extraídos dos guias de [[Playbook/playbook]]. Resolver os tokens pela [[Playbook/visao-profile]] antes de usar qualquer um.

## Código

- [[Templates/Codigo/container.vue]] — container fino do card. Guia: [[Playbook/container]].
- [[Templates/Codigo/service.js]] — service da área, dividido por recurso. Guia: [[Playbook/services]].
- [[Templates/Codigo/mapper.js]] — mappers de leitura/escrita da entidade. Guia: [[Playbook/mappers]].
- [[Templates/Codigo/store-module.js]] — types e slice da store. Guias: [[Playbook/store-types]], [[Playbook/store]].
- [[Templates/Codigo/store.test.js]] — teste unitário Vitest do slice. Guia: [[Playbook/testing]].
- [[Templates/Codigo/loader-composable]] — loader do container pai. Guia: [[Playbook/loader]].
- [[Templates/Codigo/spec.playwright]] — set canônico de teste Playwright. Guia: [[Playbook/testing]].

## Módulo (nível acima do card)

- [[Templates/Codigo/module-scaffold]] — boilerplate de pastas/arquivos pra um módulo novo. Guia: [[Playbook/new-module]].

## Convenções (sem template de código)

- [[Playbook/component-conventions]] — regras de props/atributos/comentários/imports/Storybook, sem boilerplate próprio (aplicam por cima dos templates acima).

## Referências cruzadas

- [[home]]
- [[Playbook/playbook]]
