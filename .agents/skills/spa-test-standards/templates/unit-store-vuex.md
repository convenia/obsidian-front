# Template: store / Vuex (mutations + actions)

**When to use:** only the mutations/actions/getters of a Vuex store module that carry real logic — conditional branches, data transformation/merge, or chaining of dispatches/services. Pass-through code (a mutation that only assigns the payload, an action that only calls the service and commits the result with no branching, a getter that only returns state) is **not** unit-tested; it's covered by the integration flow. If the module has no logic at all, don't create a test file for it. Mutations: call directly with a hand-built `state` and `payload` — no real Vuex. Actions: call with a fake `ctx` containing only the fields the action uses (`commit`, `dispatch`, `state`, `getters`), and mock the services via `vi.mock`.

```js
import { describe, expect, it, vi, beforeEach } from 'vitest'
import * as types from '@Feature:types'
import * as services from '@Feature:services/Something'
import store from '../index'

vi.mock('@Feature:services/Something', () => ({
  getSomething: vi.fn()
}))

describe('<Feature>/store/Something', () => {
  beforeEach(() => { vi.resetAllMocks() })

  describe('mutations', () => {
    it('should replace the list when there is no pagination', () => {
      const state = { items: [ { id: 'stale' } ] }

      store.mutations[types.SET_ITEMS](state, { items: [ { id: 'fresh' } ] })

      expect(state.items).toEqual([ { id: 'fresh' } ])
    })
  })

  describe('actions', () => {
    it('should commit the data and return it on success', async () => {
      const data = { items: [] }
      services.getSomething.mockResolvedValueOnce([ null, data ])
      const commit = vi.fn()
      const dispatch = vi.fn()

      const result = await store.actions[types.FETCH_ITEMS]({ commit, dispatch }, { id: 'x' })

      expect(commit).toHaveBeenCalledWith(types.SET_ITEMS, data)
      expect(result).toEqual([ null, data ])
    })

    it('should dispatch error feedback and not commit when the service fails', async () => {
      const error = new Error('network error')
      services.getSomething.mockResolvedValueOnce([ error, null ])
      const commit = vi.fn()
      const dispatch = vi.fn()

      const result = await store.actions[types.FETCH_ITEMS]({ commit, dispatch }, { id: 'x' })

      expect(commit).not.toHaveBeenCalled()
      expect(dispatch).toHaveBeenCalledWith('FEEDBACKS_ADD', expect.objectContaining({ type: 'error' }))
      expect(result).toEqual([ error, null ])
    })
  })
})
```

## Gotcha: the same service imported through two different paths in the source file

It's common for a store module to import the same service in two ways (e.g., `import { getX } from '@Feature:services/Something'` for some actions and `import * as service from '@/Feature/services/Something/index'` for others, both resolving to the SAME physical file). In that case, **a single `vi.mock` covering all used exports is enough** — declaring `vi.mock` for the two specifiers separately makes the second one *overwrite* the first (the module registry is the same underneath, the last `vi.mock` wins).

Symptom: a `No "x" export is defined on the mock` error pointing at the specifier you **didn't** mock last, even though you mocked `x` in the other one.

Fix: a single `vi.mock`, with all exports; import and use the namespace through both specifiers normally in the test — both will hit the same mocked object.
