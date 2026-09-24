---
tags:
  - template
  - frontend
  - vuex
date: 2026-09-24
---

# Template: store-module.js

Esqueleto de types/mutations/actions de uma area migrada. Guia: [[Playbook/store]]. Tokens: [[Playbook/visao-profile]].

```js
// store/types.js — ordem alfabetica estrita
export const <TYPE_PREFIX>_DELETE_<AREA>_<ENTITY> = '<TYPE_NS><TYPE_PREFIX>_DELETE_<AREA>_<ENTITY>'
export const <TYPE_PREFIX>_GET_<AREA>_METADATA = '<TYPE_NS><TYPE_PREFIX>_GET_<AREA>_METADATA'
export const <TYPE_PREFIX>_GET_<AREA>_SECTIONS = '<TYPE_NS><TYPE_PREFIX>_GET_<AREA>_SECTIONS'
export const <TYPE_PREFIX>_SET_<AREA>_METADATA = '<TYPE_NS><TYPE_PREFIX>_SET_<AREA>_METADATA'
export const <TYPE_PREFIX>_SET_<AREA>_SECTIONS = '<TYPE_NS><TYPE_PREFIX>_SET_<AREA>_SECTIONS'
export const <TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY> = '<TYPE_NS><TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>'
export const <TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES = '<TYPE_NS><TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES'
```

```js
// store/state.js
const state = {
  <entityA>: [],
  <entityB>: {},
  fieldsMetadata: {},   // metadado de campos, chaveado por sub-area
}
```

```js
// store/mutations.js
[types.<TYPE_PREFIX>_SET_<AREA>_METADATA]: (state, data) => {
  state.fieldsMetadata = { ...data || {} }
},
[types.<TYPE_PREFIX>_SET_<AREA>_SECTIONS]: (state, { type, data }) => {
  if (!Object.hasOwn(state, type)) return
  state[type] = Array.isArray(data) ? [ ...data ] : { ...data }
},
```

```js
// store/actions.js — uma unica action de escrita decide create vs update
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
    // refazer tambem <TYPE_PREFIX>_GET_<AREA>_METADATA pra atualizar compulsory/disabled
  }

  return [ err, data ]
},

[types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES]: async ({ dispatch }, payload) => {
  const { employeeId, <entity>Id, files = {} } = payload
  const endpoint = <entity>Id ? getUploadEndpoint(employeeId, <entity>Id, '<resource>') : null

  const filePromises = [
    ...(files.upload?.length && endpoint
      ? [ upload({ files: files.upload, endpoint }).then(u => Promise.all(u.map(({ request }) => request))) ]
      : []),
    ...(files.remove || []).map(({ id: fileId }) => deleteFile({ employeeId, fileId })),
  ]

  if (!filePromises.length) return [ null, [] ]

  const results = await Promise.allSettled(filePromises)
  if (results.some(({ status }) => status === 'rejected'))
    dispatch('FEEDBACKS_ADD', { type: 'error', message: 'Erro ao atualizar anexos.', highlighted: 'Tente novamente.' })
},
```

> **Exemplo real:** em cards System+Custom Fields o state espelha dois slices (`systemFields`/`customFields`) em vez de um `fieldsMetadata` unico, e a mutation SET escreve ambos.

Regras a nao esquecer: nunca uma action `CREATE_*` separada; nunca duas actions de arquivo separadas; ordem alfabetica estrita em todo bloco tocado.

## Referencias cruzadas

- [[Playbook/store]]
- [[Playbook/visao-profile]]
