---
tags:
  - playbook
  - frontend
  - loader
date: 2026-09-24
---

# Parent Loader e Options Wiring

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O loader e UNIFICADO pra qualquer visao e qualquer card.

## O que e

O loader do container pai, o padrao de options dependentes, a renderizacao condicional de card, e o split de `provide` entre pai e filho.

## Loader pai

`<CONTAINER_DIR>index.vue` e `<script setup>` usando `useLoaders`/`addLoader`. Ele fornece contexto generico, carrega metadado + entidades + options estaticas num unico batch de `Promise`, e possui o indicador de loading **e** o espacamento entre os cards filhos. O `<c-loader>` posicionado e o espacamento `.containers > *:not(:last-child)` (+ variante responsiva) ficam aqui, no pai — os containers filhos nao carregam margem propria, ficam finos.

Todas as leituras de entidade passam pelo orquestrador SECTIONS — nunca gets individuais espalhados fora dele.

## `provide` — pai vs filho

- **Pai (`index.vue`)** fornece so o que **todo** card filho precisa: `employeeId` (o organism faz `inject`), e `getFileUrl` quando varios cards compartilham arquivos.
- **Container filho** fornece o que **aquele card** precisa: qualquer loader de option dependente (ver abaixo), e `getFileUrl` quando so aquele card tem attachment.
- Nao jogue todo `provide` no pai.

## Options dependentes — regra generica + exemplo

Toda lista de opcao que o organism precisa e carregada pelo **container**, nunca pelo organism. Duas categorias:

- **Options ja embutidas no metadado** — cada entrada do metadado de campos ja carrega o proprio `options`, que o composable de schema do organism concatena no field. Um select simples nao precisa de nenhuma action de options nem entrada `fieldsOptions` separada — o organism resolve do payload de metadado. Adicionar um fetch de options pra um select que ja vem no payload e red flag.
- **Options genuinamente dependentes** (o valor depende de outro field, ex. cidade-por-estado) — reserve `fieldsOptions` + o loader `provide`/watch so pra esse caso.

> **Exemplo real:** um card com endereco carrega cidades via a action de options do modulo (`<TYPE_PREFIX>_OPTIONS_GET_CITIES`), lida de `mapState('<options module>')`, chamada `getCities(stateId)`. O modulo de options precisa estar registrado via `<MODULE_REG>` da visao — sem isso a action lanca "unknown action type". Use sempre o endpoint REST oficial da opcao dependente, nunca uma query GraphQL legada equivalente.

## Renderizacao condicional de card

Quando um card so aparece pra alguns registros, use `v-if="<isEligible>"`. A fonte do gate e `<CARD_GATE>` — prefira o predicate do organism quando existir; nunca compute o gate a partir de um arquivo de form da SPA.

## O que remover

- Qualquer fetch de obrigatoriedade separado do metadado principal — a obrigatoriedade agora vem no payload de metadado como `compulsory: true` por field.
- Qualquer wiring de options legado (rota/action antiga de options) e seu mock de teste — dar grep no type antes de apagar; se for compartilhado (ex. outra feature ainda dispara), manter a definicao e remover so as referencias da area migrada.

## O que o loader NAO deve fazer

- Buscar obrigatoriedade separadamente do metadado.
- Buscar um options/GraphQL separado pra fields — tudo esta no fetch de metadado.
- Computar inline um valor que ja existe como action de dependent-option da store — fornecido (`provide`) pelo container **filho**.

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/store]]
- [[Playbook/container]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/loader-composable]]
