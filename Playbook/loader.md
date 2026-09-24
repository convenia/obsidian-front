---
tags:
  - playbook
  - frontend
  - loader
date: 2026-09-24
---

# Parent Loader e Options Wiring

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O loader e UNIFICADO pra qualquer visao.

## O que e

O loader do container pai, o padrao de options dependentes (ex.: `getCities`), a renderizacao condicional de card, e o split de `provide` entre pai e filho.

## Loader pai

`<CONTAINER_DIR>index.vue` e `<script setup>` usando `useLoaders`/`addLoader`. Ele fornece contexto generico, carrega fields + entidades + options estaticas num unico batch de `Promise`, e possui o indicador de loading **e** o espacamento entre os cards filhos. O `<c-loader>` posicionado e o espacamento `.containers > *:not(:last-child)` (+ variante responsiva) ficam aqui, no pai — os containers filhos nao carregam margem propria, ficam finos.

Todas as leituras de entidade passam pelo orquestrador SECTIONS — nunca gets individuais espalhados fora dele.

## `provide` — pai vs filho

- **Pai (`index.vue`)** fornece so o que **todo** card filho precisa: `employeeId` (o organism faz `inject`), e `getFileUrl` quando varios cards compartilham arquivos.
- **Container filho** fornece o que **aquele card** precisa: `getCities` e `getZipCode` (encapsulando as actions da store), e `getFileUrl` quando so aquele card tem attachment.
- Nao jogue todo `provide` no pai.

## Padrao `getCities`

Toda lista de opcao que o organism precisa e carregada pelo **container**, nunca pelo organism.

- **Estados** — carregado no mesmo batch de `Promise` via `commonTypes.STATES`; lido de `mapState('cityState')`.
- **Cidades (dinamico)** — via a action do modulo de options `<TYPE_PREFIX>_OPTIONS_GET_CITIES`; lido de `mapState('<options module>')`.

O modulo de options precisa estar registrado via `<MODULE_REG>` da visao — sem isso a action lanca "unknown action type". Use o endpoint REST `<CITIES_EP>`; nunca a GraphQL legada `GetCitiesByState`.

## Por que a options-em-payload existe

Cada entrada de `systemFields` carrega o proprio `options`, que `createSystemFieldsSchema` concatena no schema do field. Um select simples nao precisa de nenhuma action de options nem entrada `fieldsOptions` — o organism resolve do payload. Reserve `fieldsOptions` + o loader `provide`/watch so pra options genuinamente dependentes (um valor que depende de outro field, ex. cidade-por-estado). Adicionar um fetch de options pra um select que ja vem no payload e red flag.

## Renderizacao condicional de card

Quando um card so aparece pra alguns colaboradores, use `v-if="<isEligible>"`. A fonte do gate e `<CARD_GATE>` — prefira o predicate do organism quando existir; nunca compute o gate a partir de um arquivo de form da SPA.

## O que remover

- `getRequiredSystemFields` / `<TYPE_PREFIX>_GET_REQUIRED_SYSTEM_FIELDS` — a obrigatoriedade agora vem no payload de fields como `compulsory: true` por field.
- Qualquer wiring de options legado (`<TYPE_PREFIX>_<AREA>_OPTIONS` / GraphQL `getOptions`) e seu mock de teste — dar grep no type antes de apagar; se for compartilhado (ex. Admission ainda dispara), manter a definicao e remover so as referencias da area migrada.

## O que o loader NAO deve fazer

- Buscar `requiredSystemFields`.
- Buscar um options/GraphQL separado pra fields — tudo esta no fetch de fields.
- Computar CEP inline — autofill de CEP e a action de store, fornecida (`provide`) pelo container **filho**.

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/store]]
- [[Playbook/container]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/loader-composable]]
