# Template: containers/components via `shallowMount` (behavior tied to the template)

**When to use:** the test depends on something that only exists after the template renders: a prop passed to a child, a conditional class, an element count, an event emitted by a stubbed child.

General rules:
- Mock modules with side effects (router, composition helpers, store) via `vi.mock` at the top of the file, before importing the component under test.
- `shallowMount` is the default (not `mount`) — it stubs children.
- Library/organism component stub: `true` when only presence matters; a minimal `{ template, name, props }` object when you need to trigger events (`wrapper.findComponent({name}).vm.$emit(...)`) or read received props (`wrapper.findComponent({name}).props('field')`).
- Assertions via `wrapper.findComponent`, `wrapper.emitted()`, `wrapper.vm.<data|computed>` — don't use `getByTestId`/`getByRole` here if the `@testing-library` package isn't installed in the project; the common pattern in these SPAs is plain Vue Test Utils.
- **Always check `wrapper.findComponent({ name })` against the real import/registration name** of the `.vue` under test — a `stubs: { WrongName: ... }` that doesn't match the real name "passes" only by accident (`shallowMount`'s auto-stub stubs the real import anyway), not because the declared stub is actually in use.

## Base template (Composition API)

```js
import { describe, it, expect, vi } from 'vitest'
import { shallowMount } from '@vue/test-utils'
import { useActions, useGetters } from 'vuex-composition-helpers'
import MyContainer from '../MyContainer.vue'

vi.mock('vuex-composition-helpers', () => ({
  useActions: vi.fn(),
  useGetters: vi.fn()
}))

const buildWrapper = (overrides = {}) => {
  useActions.mockReturnValue({ fetchSomething: vi.fn().mockResolvedValue(undefined), ...overrides.actions })
  useGetters.mockReturnValue({ items: [], isLoading: false, ...overrides.getters })

  return shallowMount(MyContainer)
}

describe('MyContainer', () => {
  it('should render the empty state when there are no items', () => {
    const wrapper = buildWrapper({ getters: { items: [] } })

    expect(wrapper.find('[data-testid="empty-state"]').exists()).toBe(true)
  })
})
```

## Mutation-proofing: in-flight request guard

"Don't trigger a second request while one is already in flight": simulate a promise that never resolves, fire the trigger event twice, and assert `toHaveBeenCalledTimes(1)`. Without this, a mutation that removes the "request in flight" guard slips through:

```js
it('should not trigger a second fetch while a request is still in flight', async () => {
  const fetchSomething = vi.fn().mockImplementation(() => new Promise(() => {})) // never resolves
  const wrapper = buildWrapper({ actions: { fetchSomething } })

  await wrapper.vm.onScrollLimitReached()
  await wrapper.vm.onScrollLimitReached()

  expect(fetchSomething).toHaveBeenCalledTimes(1)
})
```

## Mutation-proofing: stub with programmatic emit to validate client-side wiring

When the component under test orchestrates a field's validation (e.g., a shared library's form builder), stub the child with a `template` that emits the expected event on a simulated click, and assert the resulting props/state before and after — no need to mount the library's real form to prove the container's validation wiring is correct:

```js
const CFormBuilderStub = {
  name: 'CFormBuilder',
  props: ['schema', 'value'],
  template: '<button data-testid="stub-submit" @click="$emit(\'submit\', { valid: false })" />'
}

it('should keep the modal open and show the validation error when the form is invalid', async () => {
  const wrapper = shallowMount(MyContainer, {
    stubs: { CFormBuilder: CFormBuilderStub }
  })

  await wrapper.find('[data-testid="stub-submit"]').trigger('click')

  expect(wrapper.emitted('close')).toBeUndefined()
  expect(wrapper.vm.hasValidationError).toBe(true)
})
```
