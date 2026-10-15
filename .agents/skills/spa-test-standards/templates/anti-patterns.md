# Common test anti-patterns (don't do this)

Findings that recur across tests that "pass today" but a good portion wouldn't catch a real regression. If you recognize one of these while writing or reviewing a test, stop and fix it before moving on.

## 1. Generic `Proxy` mock that returns the same function/value for any key

`useActions`/`useGetters`/`useMutations` mocked as `new Proxy({}, { get: () => vi.fn() })` (or `get: () => ref([])`) make it impossible to verify *which* action was called with *which* payload — every assertion becomes tautological.

```js
// ❌ Anti-pattern — generic Proxy
vi.mock('vuex-composition-helpers', () => ({
  useActions: () => new Proxy({}, { get: () => vi.fn() }),
  useGetters: () => new Proxy({}, { get: () => ref([]) })
}))
// no assertion can prove the exact payload of `onSelectItem`,
// and `permissions.includes(anything)` is always `false` — the test passes even with the wrong permission

// ✅ Correct — each key mocked individually and named
const onSelectItem = vi.fn()
useActions.mockReturnValue({ onSelectItem })
useGetters.mockReturnValue({ permissions: [ 'edit_item' ] })
```

**Rule:** mock each used key individually as a named `vi.fn()`, even if it means more setup lines.

## 2. Pinning a `v-if` flag to the same value always

Pinning `isLoading`/`isPaginating` (or any flag used in `v-if`) always to `false` (or always to `true`) in the composable's mock. The opposite branch of the template is never exercised — a `v-if="isLoading"` that renders always, or never, doesn't break any test.

```js
// ❌ Anti-pattern — every test in the file uses the same fixed value
useLoaders.mockReturnValue({ isLoading: ref(false) })

// ✅ Correct — at least one test sets the opposite value and checks the effect
it('should show the loading skeleton while items are loading', () => {
  useLoaders.mockReturnValue({ isLoading: ref(true) })
  const wrapper = buildWrapper()

  expect(wrapper.find('[data-testid="loading-skeleton"]').exists()).toBe(true)
})
```

**Rule:** at least one test per component should set the opposite of the default value and check the effect (conditional render or guard).

## 3. Testing only one side of a result guard/branch

Different from the general principle "test both sides of every conditional" (which covers execution guards), here the gap is specifically leaving only the ERROR path tested and never the SUCCESS path (or vice versa) in a flow that has logic on both sides — e.g., only a test exists for the failure branch of a confirmation, and the success branch (clearing state, redirecting with the right id) has no test at all, so a regression in the success redirect goes unnoticed.

See the full example (both branches tested) in `templates/unit-container-no-mount.md`, section "Mutation-proofing: test both sides of every result branch".

## 4. Presence assertion when the exact value is known

`toContain(substring)`, `toHaveLength(n)`, `toBeDefined()` pass even if another part of the value is wrong.

```js
// ❌ Anti-pattern — swapping any other part of the sentence doesn't break the test
expect(message).toContain(startDate)

// ✅ Correct — the full expected string
expect(message).toBe(`The period starts on ${startDate} and ends on ${endDate}`)
```

## 5. Leaving the riskiest logic in the file untested

A sign that effort went into what was easy to mock, not what was risky to break — e.g., a single test covering a trivial simple field passthrough, while a 200+ line action-by-status mapping function with multiple conditional spreads (`canEdit`, `hasPermissionX`) has no test at all, and the stub used in the component doesn't even expose that logic for the test to reach.

**Rule:** when reviewing a test file, ask "which function/branch has the most conditional logic here — does it have a test, or only the simplest one does?".

## 6. Testing only the extracted pure function and forgetting the component that consumes it

The pure function can be 100% covered while the container component that decides *when* to call it is at 0% — e.g., the exported validation functions from a module have full test coverage, but the sibling `.vue` that uses them (submit guard, recompute on `onInput`, prefill on `onMounted`) has no test file at all.

**Rule:** when adding a unit test for a pure function extracted from a component, confirm the component that consumes it also has coverage (via `templates/unit-container-no-mount.md` or `templates/unit-container-shallow-mount.md`, as appropriate).

## 7. Stub name that doesn't match the real name of the imported component

`stubs: { GenericName: ... }` when the component is imported/registered under a different name makes the test "pass" only by accident (`shallowMount`'s auto-stub stubs the real import anyway), not because the declared stub is actually in use.

```js
// ❌ Anti-pattern — the real component is named "ConfirmationModal", not "CommonConfirmationModal"
shallowMount(MyContainer, { stubs: { CommonConfirmationModal: true } })

// ✅ Correct — check the real import/registration name before declaring the stub
shallowMount(MyContainer, { stubs: { ConfirmationModal: true } })
expect(wrapper.findComponent({ name: 'ConfirmationModal' }).exists()).toBe(true)
```

**Rule:** always check `wrapper.findComponent({ name })` against the real import/registration name of the `.vue` under test, don't assume the `stubs` key matched.
