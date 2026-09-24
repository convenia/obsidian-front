---
tags:
  - dashboard
  - segundo-cerebro
  - frontend
  - vue
date: 2026-09-24
aliases:
  - inicio
---

# Home

Este vault funciona como um segundo cerebro operacional para orientar devs e IA na arquitetura frontend das SPAs Convenia — o padrao generico de card que consome um organism de `@convenia/components` (metadado de campos + entidade, container fino, store Vuex, testes Playwright), aplicavel a qualquer dominio, nao so a um card especifico.

> [!info] Fluxo principal
> Use [[Playbook/playbook]] pra entender o padrao e [[Templates/templates]] pra gerar codigo a partir dele.

## Navegacao

- [[Agents/agents]]
- [[Playbook/playbook]]
- [[Templates/templates]]
- [[Projects/projects]]
- [[Tools/tools]]

## Como comecar uma migracao de card

1. Ler [[Playbook/visao-profile]] e resolver os tokens pra sua visao (spa-admin, spa-colab-self, spa-colab-supervisor).
2. Seguir a ordem dos guias: [[Playbook/services-and-mappers]] → [[Playbook/store]] → [[Playbook/loader]] → [[Playbook/container]] → [[Playbook/testing]] → [[Playbook/cleanup]].
3. Usar os templates de [[Templates/templates]] como ponto de partida de cada camada.
4. Conferir o resultado contra [[Playbook/red-flags]] antes de considerar a migracao pronta.

## Origem do conteudo

O padrao documentado aqui foi destrinchado das skills `spas-reference` e `system-custom-fields-organism` (submodule `.@convenia` do repo `spa-colab`, path `src/organisms/Employee/SystemFields/specs/skills/`), que documentam a migracao de cards System+Custom Fields. Este vault generaliza esse conteudo pra qualquer card que consome um organism compartilhado — System+Custom Fields aparece so como "Exemplo real" pontual dentro dos guias, nunca como o vocabulario padrao das regras.
