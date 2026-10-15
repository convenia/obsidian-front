# Template: Options API containers (Vue 2) — without mounting

**When to use:** the component has no relevant **template** logic for the case at hand (no conditional class, slot, or prop passthrough in play). It's cheaper and more direct to test `methods`/`computed` without `shallowMount`: import the component, destructure `{ methods, computed }`, and call them with `.call(fakeCtx, ...args)`. No real Vuex, no render resolution.

```js
import { describe, it, expect, vi } from 'vitest'
import MyContainer from '../MyContainer.vue'

const { methods, computed } = MyContainer

describe('MyContainer', () => {
  describe('load', () => {
    it('should redirect when the fetch fails', async () => {
      const ctx = {
        $route: { params: { id: 'x' } },
        $router: { push: vi.fn() },
        getSomething: vi.fn().mockResolvedValue([ new Error('fail') ])
      }

      await methods.load.call(ctx)

      expect(ctx.$router.push).toHaveBeenCalledWith({ name: 'ERROR_ROUTE' })
    })
  })

  describe('someComputed', () => {
    it('should derive the label from state', () => {
      expect(computed.someComputed.call({ status: 'active' })).toBe('Active')
    })
  })
})
```

## Gotcha: a computed that reads another computed from the same component

E.g., `character()` reading `this.isProcessing`, which is also a computed — in a plain `ctx` this isn't resolved automatically the way Vue would at runtime. Pass the already-computed value directly in the `ctx` (`{ isProcessing: true }`), don't rely on the real chaining — or test the dependent computed in isolation with its own `ctx`.

## Mutation-proofing: test both sides of every result branch

It's not enough to test only the error branch (or only the success branch) when both have their own logic — a regression on the untested side slips through:

```js
describe('onConfirmDuplication', () => {
  it('should keep the item set when duplication fails', async () => {
    const ctx = {
      itemToDuplicate: { id: 'x' },
      duplicateItem: vi.fn().mockResolvedValue([ new Error('fail') ]),
      $router: { push: vi.fn() }
    }

    await methods.onConfirmDuplication.call(ctx)

    expect(ctx.itemToDuplicate).toEqual({ id: 'x' })
    expect(ctx.$router.push).not.toHaveBeenCalled()
  })

  it('should clear the item and redirect with the new id on success', async () => {
    const ctx = {
      itemToDuplicate: { id: 'x' },
      duplicateItem: vi.fn().mockResolvedValue([ null, { id: 'new-id' } ]),
      $router: { push: vi.fn() }
    }

    await methods.onConfirmDuplication.call(ctx)

    expect(ctx.itemToDuplicate).toBeNull()
    expect(ctx.$router.push).toHaveBeenCalledWith({ name: 'DETAIL', params: { id: 'new-id' } })
  })
})
```
