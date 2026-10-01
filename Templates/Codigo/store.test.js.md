---
tags:
  - template
  - frontend
  - vitest
date: 2026-10-01
---

# Template: store.test.js

Boilerplate do teste unitário de um slice Vuex em Vitest. O teste unitário cobre só fluxo de erro da action; o caminho feliz fica no E2E ([[Templates/Codigo/spec.playwright]]). Guia: [[Playbook/testing]], seção "Teste unitário de store (Vitest)". Slice testado: [[Templates/Codigo/store-module.js]]. Tokens: [[Playbook/visao-profile]].

```js
// store/<Feature>/tests/index.test.js
import { describe, it, expect, vi, beforeEach } from 'vitest'
import * as types from '<ROOT_ALIAS>:types'
import * as services from '<ROOT_ALIAS>:services/<Area>/<Feature>'
import store from '../index'

vi.mock('<ROOT_ALIAS>:services/<Area>/<Feature>', () => ({
  create<Entity>: vi.fn(),
  get<Entities>: vi.fn(),
  update<Entity>: vi.fn(),
}))

const { actions } = store

describe('<ROOT>/store/<Feature>', () => {
  beforeEach(() => {
    vi.resetAllMocks()
  })

  describe(types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>, () => {
    it('should dispatch error feedback and not commit on error', async () => {
      const error = new Error('fail')
      services.get<Entities>.mockResolvedValueOnce([ error, null ])
      const commit = vi.fn()
      const dispatch = vi.fn()

      const result = await actions[types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>]({ commit, dispatch }, { employeeId: 'employee-1' })

      expect(commit).not.toHaveBeenCalled()
      expect(dispatch).toHaveBeenCalledWith('FEEDBACKS_ADD', {
        type: 'error',
        message: '<Entidades>',
        highlighted: 'não foram carregadas.'
      })
      expect(result).toEqual([ error, null ])
    })
  })

  describe(types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>, () => {
    it('should dispatch error feedback and not refetch on error', async () => {
      const error = new Error('fail')
      services.update<Entity>.mockResolvedValueOnce([ error, null ])
      const dispatch = vi.fn()

      const result = await actions[types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]({ dispatch }, { employeeId: 'employee-1', id: '1' })

      expect(dispatch).not.toHaveBeenCalledWith(types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>, expect.anything())
      expect(dispatch).toHaveBeenCalledWith('FEEDBACKS_ADD', {
        type: 'error',
        message: '<Entidade>',
        highlighted: 'não foi salva.'
      })
      expect(result).toEqual([ error, null ])
    })
  })
})
```

## Referências cruzadas

- [[Playbook/testing]]
- [[Playbook/store]]
- [[Templates/Codigo/store-module.js]]
- [[Templates/Codigo/spec.playwright]]
