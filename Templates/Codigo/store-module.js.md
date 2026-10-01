---
tags:
  - template
  - frontend
  - vuex
date: 2026-10-01
---

# Template: store-module.js

Boilerplate de um slice Vuex: types e slice. Guias: [[Playbook/store-types]], [[Playbook/store]]. Tokens: [[Playbook/visao-profile]]. Teste: [[Templates/Codigo/store.test.js]]. Registro do slice: [[Templates/Codigo/module-scaffold]].

## Types

```js
// store/types.js — agrupado por feature, ordem alfabética estrita dentro do grupo
export * from '@types'

// <Feature>
export const <TYPE_PREFIX>_<FEATURE>_DELETE_<AREA> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_DELETE_<AREA>'
export const <TYPE_PREFIX>_<FEATURE>_GET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_GET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_HAS_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_HAS_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_RESET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_RESET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_SET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_SET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>'
```

## Slice base

```js
// store/<Feature>/index.js
import * as types from '<ROOT_ALIAS>:types'
import * as services from '<ROOT_ALIAS>:services/<Area>/<Feature>'

const state = {
  <entities>: [],
}

const getters = {
  [types.<TYPE_PREFIX>_<FEATURE>_HAS_<AREAS>]: ({ <entities> }) => !!<entities>.length,
}

const mutations = {
  [types.<TYPE_PREFIX>_<FEATURE>_RESET_<AREAS>]: (state) => {
    state.<entities> = []
  },

  [types.<TYPE_PREFIX>_<FEATURE>_SET_<AREAS>]: (state, data) => {
    state.<entities> = [ ...(data || []) ]
  },
}

const actions = {
  [types.<TYPE_PREFIX>_<FEATURE>_DELETE_<AREA>]: async ({ dispatch }, { employeeId, id } = {}) => {
    const [ err, data ] = await services.delete<Entity>({ employeeId, id })

    if (!err) await dispatch(types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>, { employeeId })

    dispatch('FEEDBACKS_ADD', {
      type: err ? 'error' : 'success',
      message: '<Entidade>',
      highlighted: err ? 'não foi removida.' : 'removida com sucesso.'
    })

    return [ err, data ]
  },

  [types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>]: async ({ commit, dispatch }, { employeeId } = {}) => {
    const [ err, data ] = await services.get<Entities>({ employeeId })

    if (!err) commit(types.<TYPE_PREFIX>_<FEATURE>_SET_<AREAS>, data)

    if (err) dispatch('FEEDBACKS_ADD', {
      type: 'error',
      message: '<Entidades>',
      highlighted: 'não foram carregadas.'
    })

    return [ err, data ]
  },

  // Só quando o recurso tem create e update. Com só um dos dois, chame o service direto.
  [types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]: async ({ dispatch }, payload) => {
    const { employeeId, id } = payload || {}
    const isCreating = !id

    const [ err, data ] = isCreating
      ? await services.create<Entity>(payload)
      : await services.update<Entity>(payload)

    if (!err) await dispatch(types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>, { employeeId })

    dispatch('FEEDBACKS_ADD', {
      type: err ? 'error' : 'success',
      message: '<Entidade>',
      highlighted: err ? 'não foi salva.' : 'salva com sucesso.'
    })

    return [ err, data ]
  },
}

export default { state, getters, mutations, actions }
```

Assignature longa demais pro limite de linha do ESLint:

```js
[types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]: async (context, payload) => {
  const { dispatch, state } = context || {}
  const { employeeId, id, ...data } = payload || {}
  // ...
},
```

Regras a não esquecer: types na fórmula `<TYPE_PREFIX>_<FEATURE>_<VERB>_<AREA>`; action e mutation com types distintos; state em ordem alfabética e só com valor vazio; sempre `return [ err, data ]`; sem `try/catch` na action; um único `FEEDBACKS_ADD` por fluxo, com texto comum fixo; refetch dentro da action.

## Referências cruzadas

- [[Playbook/store-types]]
- [[Playbook/store]]
- [[Playbook/visao-profile]]
- [[Playbook/services]]
- [[Templates/Codigo/store.test.js]]
- [[Templates/Codigo/module-scaffold]]
