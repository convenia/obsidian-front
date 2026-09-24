---
tags:
  - playbook
  - frontend
  - vuex
date: 2026-09-24
---

# Store Slice e Types

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O shape de store e UNIFICADO pra qualquer visao — e pra qualquer card, nao so um dominio especifico.

## O que e

O state, os type constants, as mutations e as actions da store Vuex de uma area migrada.

## Shape do state

```js
const state = {
  <entityA>: [],       // slices de entidade existentes, sem mudanca
  <entityB>: {},
  fieldsMetadata: {},  // metadado de campos do card, chaveado por sub-area
}
```

> **Exemplo real:** em cards System+Custom Fields o metadado se divide em dois slices espelhados, `systemFields` e `customFields` (mesmas chaves de sub-area, mesmo ciclo de vida). Um card sem essa distincao usa um unico slice `fieldsMetadata`.

## Naming (convencao canonica)

Todo type constant coloca o prefixo da feature primeiro, depois o verbo: `<TYPE_PREFIX>_<VERB>_[<AREA>_]<ENTITY>`. Ao tocar uma area que ainda carrega types legados `VERB_<TYPE_PREFIX>_*`, renomeie pra essa convencao na mesma mudanca — depois de dar grep no repo inteiro pra atualizar todos os dispatchers.

## Ordering (convencao canonica)

`state`, `mutations`, `actions` e `types.js` ficam em ordem alfabetica estrita pelo nome do constant. Ao tocar um arquivo, reordene o bloco inteiro — inclusive entradas ja existentes, nunca so anexe no final.

## Types

Um par METADATA (uma action busca o metadado do card; uma mutation escreve o slice) + um par SECTIONS orquestrador (as leituras de entidade passam por ele):

```js
export const <TYPE_PREFIX>_GET_<AREA>_METADATA = '<TYPE_NS><TYPE_PREFIX>_GET_<AREA>_METADATA'
export const <TYPE_PREFIX>_SET_<AREA>_METADATA = '<TYPE_NS><TYPE_PREFIX>_SET_<AREA>_METADATA'
export const <TYPE_PREFIX>_GET_<AREA>_SECTIONS = '<TYPE_NS><TYPE_PREFIX>_GET_<AREA>_SECTIONS'
export const <TYPE_PREFIX>_SET_<AREA>_SECTIONS = '<TYPE_NS><TYPE_PREFIX>_SET_<AREA>_SECTIONS'
```

Remova qualquer type legado que buscava metadado em pedacos separados (uma rota/action por sub-recorte do metadado).

Card com attachment adiciona um trio de arquivo: `<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES` substitui os dois types de upload/delete separados.

## Uma unica action de escrita, que decide create vs update

Uma unica action `<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>` serve create e update, tanto pra card single-item quanto multi-item:

```js
[types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>]: async ({ dispatch, state }, payload) => {
  const { employeeId, ...rest } = payload || {}
  const metadata = state.fieldsMetadata?.<sub> || {}   // NAO: const { metadata } = ...
  const isCreating = !payload.id
  const params = { employeeId, metadata, ...rest }

  const [ err, data ] = isCreating
    ? await services.create<Entity>(params)
    : await services.update<Entity>(params)

  dispatch('FEEDBACKS_ADD', { /* sucesso/erro, derivado de isCreating */ })

  if (!err) {
    await dispatch(types.<TYPE_PREFIX>_GET_<ENTITY>, { employeeId })
    // tambem refazer o fetch de <TYPE_PREFIX>_GET_<AREA>_METADATA pra atualizar compulsory/disabled
  }

  return [ err, data ]
}
```

- Decide via `!payload.id` — nunca adicione uma action `CREATE_*` separada, e o container nunca escolhe o branch (ver [[Playbook/container]]).
- No sucesso, refaz o GET da entidade especifica **e** o GET de metadado da area (pra atualizar `compulsory`/`disabled`).

## Action de arquivo

Uma unica action (`<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES`) roda uploads e removes juntos via `Promise.allSettled` e dispara `FEEDBACKS_ADD` em qualquer rejeicao. Substitui os dois types/actions legados de upload/delete separados — remova ambos.

## O que remover ao migrar

- Qualquer resolucao de valor de campo dinamico feita na mutation SET — o organism resolve isso internamente numa sub-area migrada.
- Leituras de entidade espalhadas fora do orquestrador SECTIONS.

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/services-and-mappers]]
- [[Playbook/loader]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/store-module.js]]
