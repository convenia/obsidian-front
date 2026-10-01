---
tags:
  - dashboard
  - segundo-cerebro
  - frontend
  - vue
date: 2026-10-01
aliases:
  - inicio
---

# Home

Este vault funciona como um segundo cérebro operacional para orientar devs e IA na arquitetura frontend das SPAs Convenia — o padrão genérico de card que consome um organism de `@convenia/components` (metadado de campos + entidade, container fino, store Vuex, testes Playwright), aplicável a qualquer domínio, não só a um card específico.

> [!info] Fluxo principal
> Use [[Playbook/playbook]] pra entender o padrão e [[Templates/templates]] pra gerar código a partir dele.

## Navegação

- [[README]] — instalação como referência pra Claude/Codex
- [[Agents/agents]]
- [[Playbook/playbook]]
- [[Templates/templates]]
- [[Projects/projects]]
- [[Tools/tools]]

## Como começar uma migração de card

1. Ler [[Playbook/visao-profile]] e resolver os tokens pra sua visão (spa-admin, spa-colab-self, spa-colab-supervisor).
2. Seguir a ordem dos guias: [[Playbook/services]] → [[Playbook/mappers]] → [[Playbook/store-types]] → [[Playbook/store]] → [[Playbook/loader]] → [[Playbook/container]] → [[Playbook/testing]] → [[Playbook/cleanup]].
3. Usar os templates de [[Templates/templates]] como ponto de partida de cada camada.
4. Conferir o resultado contra [[Playbook/red-flags]] antes de considerar a migração pronta.

## Como começar um módulo novo

Assunto separado (nível de módulo, não de card) — ver [[Playbook/new-module]] e o boilerplate em [[Templates/Codigo/module-scaffold]].

## Origem do conteúdo

O padrão documentado aqui foi destrinchado das skills `spas-reference` e `system-custom-fields-organism` (submodule `.@convenia` do repo `spa-colab`, path `src/organisms/Employee/SystemFields/specs/skills/`), que documentam a migração de cards System+Custom Fields. Este vault generaliza esse conteúdo pra qualquer card que consome um organism compartilhado — System+Custom Fields aparece só como "Exemplo real" pontual dentro dos guias, nunca como o vocabulário padrão das regras.
