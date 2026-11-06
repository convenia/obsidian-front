---
name: spa-test-standards
description: >
  Senior testing standard for Convenia Vue SPAs: how to decide between unit
  tests (Vitest) and integration tests (Playwright), canonical generic
  templates per layer in templates/ (pure function, store/Vuex with logic,
  Options API container without mount, container/component via shallowMount,
  Page Object Model and Playwright spec, plus a MirageJS REST mock server
  template for features that ship src/<Feature>/mocks/ on miragejs),
  selector rules (getByTestId/getByRole, never waitForTimeout/setTimeout),
  and mutation-testing mindset (every assertion must catch a real behavior
  change, not just execute the line). Use when writing, reviewing, splitting,
  or refactoring Playwright specs in tests/specs/** or unit tests in
  src/**/tests/** in any Vue feature/project at Convenia — the templates are
  generic and applicable to any Convenia SPA, not tied to a specific feature
  or repo. Also decides, per feature, whether integration mocks should go
  through MirageJS or plain page.route/mockFromList, and flags MirageJS-
  specific issues (dead models/factories, hardcoded routes with no failure
  branch, competing page.route for a Mirage-served endpoint) — see "Mocking
  backend for integration tests". When a task asks to migrate a feature's
  REST mocks to MirageJS (or to add Mirage to a feature), it invokes the
  peer skill `convenia-mirage-mocks:mirage-mocks` for the src/<Feature>/mocks
  scaffolding and keeps ownership of the Playwright side (POM, fixtures,
  storageKey, spec) — see "Migrating a feature's mocks to MirageJS". Also validates a PR's diff coverage
  against the team's minimum threshold and writes the missing unit/integration
  tests when it falls short — see "Coverage gate". Also audits a branch/PR's
  test quality (not just its coverage number) and produces a findings list
  gated per-item behind a user choice to fix/skip/discard before writing
  anything — see "Review mode". Also invocable directly via /test-standards;
  carries a self-updating field-notes log (templates/field-notes.md) of
  lessons learned across past sessions, gated by user approval before
  anything is written — see "Continuous learning loop".
metadata:
  category: testing
  version: "3.6.1"
---

# SPA Test Standards (Convenia)

## When to use
- Writing or reviewing a Playwright spec (integration) or a Vitest unit test.
- Bootstrapping a new feature's test suite from scratch, replicating a canonical pattern (see `templates/`).
- Deciding whether an existing integration test should become a unit test, or whether a new unit test makes an integration test redundant.
- Splitting a large/slow spec file into smaller files, or extracting a helper shared between specs.
- An integration test fails and it's unclear whether it's a real regression or an environment issue.
- Reviewing a test PR and assessing whether the assertions would actually catch a regression, not just "pass today".
- Every task performed needs to validate test adjustments or write new ones.
- Validating whether a branch/PR meets the team's minimum diff-coverage
  threshold before it's opened/merged, and writing the missing tests when it
  doesn't — see "Coverage gate".
- Auditing the tests already in a branch/PR for quality (mutation-safety, not
  just line coverage) and deciding, item by item, which findings to actually
  act on — see "Review mode".

## Continuous learning loop (read + propose + write, gated by your approval)
This skill accumulates real field experience across sessions instead of staying
static — that's what makes it a senior standard rather than a fixed checklist.
`templates/field-notes.md` is a living, append-only log; treat its entries as
carrying the same authority as the permanent sections below, just not yet
consolidated into them. Nothing is written to `field-notes.md` or `SKILL.md`
without your explicit sign-off first — a wrong or noisy "lesson" that slips in
unreviewed is worse than no lesson at all.

1. **Read first.** At the start of any task this skill handles, read
   `templates/field-notes.md`. Apply its entries alongside the rules in this
   file — an un-consolidated field note is still a rule, not a suggestion.
2. **Draft, don't write, at the end.** Before finishing, ask: did this task
   surface a genuinely new, non-obvious, reusable lesson — a flakiness root
   cause, a decision-rule edge case, a mock/selector gotcha, a correction you
   gave about how testing should be done at Convenia? If yes, draft the exact
   dated bullet (generalized: no feature names, ticket/PR numbers) but do
   **not** append it yet. If it's a one-off detail specific to a single
   feature with no reusable technique, don't draft anything — this log is for
   durable technique, not per-feature trivia.
3. **Halt at a User-Input Gate before writing anything.** Present the drafted
   bullet(s) — and, if 3+ notes now point at the same underlying issue, the
   proposed consolidation into a permanent `SKILL.md` section — and invoke a
   User-Input Gate (`AskUserQuestion` on Claude Code) with options such as
   "Aprovar como está" / "Editar" / "Descartar". This is a tool call, not a
   numbered list in text — see the `user-input-gates` skill's User Interaction
   Protocol. Only after explicit approval:
   - Append the (possibly edited) bullet to `field-notes.md` in its documented
     format.
   - If consolidating, fold the pattern into the relevant permanent section
     (anti-patterns, decision rules, diagnosis, mock isolation, etc.), delete
     the superseded raw entries from `field-notes.md`, and bump
     `metadata.version` in this file's frontmatter (patch for a plain
     field-note addition, minor when a consolidation promotes something into
     a permanent section, or when the coverage-gate behavior itself changes).
   - If the user edits or discards the draft instead, use exactly what they
     approved (or write nothing) — never fall back to your own original
     wording after they've changed it.
4. **Also invocable directly** via `/test-standards` (force-load this skill
   for the current task) and `/test-standards-learn "<lesson>"` (draft a
   lesson immediately from your own words, without waiting for a task's
   natural end — still gated by the same approval step before it's written).

## Mindset: a test only counts if it catches a mutation
Line coverage proves the code **ran**, not that the **right value was verified**. When writing or reviewing a test, ask: *"if someone swapped this `&&` for `||`, this `>` for `>=`, inverted this `if`, swapped this value for another, or deleted this early `return` — would any test break?"* If the answer is no, the test gives false confidence. Practical checklist:

1. **Test both sides of every conditional**, not just the happy path. A removed/inverted guard is only caught if a test exists for each side. → see `templates/unit-container-shallow-mount.md` (in-flight request guard).
2. **Assert the exact value, not just presence.** Prefer `toBe`/`toEqual` with the actual expected value over `toBeTruthy()`/`toBeDefined()`/`toBeGreaterThan(0)` when the exact value is known — presence doesn't catch a wrong value.
3. **Verify the exact arguments of every relevant call.** `toHaveBeenCalledWith(fullObject)`, not just `toHaveBeenCalled()` — a call with the wrong object still "passes" the second check.
4. **Explicitly test the "should NOT call" case** as much as the "should call" case. Most real regressions are a removed guard or early return, and that only surfaces if there's an `expect(fn).not.toHaveBeenCalled()` in the scenario where the call shouldn't happen. → see `templates/unit-container-shallow-mount.md` (in-flight request guard).
5. **Cover boundary values**, not just a "generic case": `0`, negative, above the allowed maximum, empty string, empty array, `undefined`/`null`, one item vs. multiple items. → see `templates/unit-pure-function.md`.
6. **When logic aggregates/filters by flags**, test that turning ONE flag off actually excludes the corresponding field from the result — don't just test "with everything on". → see `templates/unit-pure-function.md`.
7. **A whole return object → `toEqual(fullObject)`**, not three loose `toBe` calls on the fields you "remembered to test". This catches mutations in fields nobody thought to check individually.
8. This doesn't mean inflating the `it()` count: two related `expect()` calls that validate facets of the same result belong in the same test. The criterion is "does this test fail if the behavior changes", not "how many tests do I have".

## Common anti-patterns (don't do this)
Recurring findings in tests that "pass today" but a good portion wouldn't catch a real regression. If you recognize one of these while writing or reviewing a test, stop and fix it before moving on — details and before/after examples in `templates/anti-patterns.md`:

1. Generic `Proxy` mock that returns the same function/value for any key (`useActions`/`useGetters` as `new Proxy({}, { get: () => vi.fn() })`).
2. Pinning a `v-if` flag (`isLoading`, `isPaginating`) to the same value always — the opposite branch is never exercised.
3. Testing only one side of a result guard/branch (only the error path, or only the success path).
4. Presence assertion (`toContain`, `toBeDefined`) when the exact value is known and simple to write.
5. Leaving the riskiest logic in the file untested, while the trivial path has a test.
6. Testing only the pure function extracted from a component and forgetting the `.vue` that consumes it.
7. Stub name that doesn't match the real name of the imported component.
8. A MirageJS model/factory/seed created for a feature but no route ever reads from `schema.db` for it (a "dead mock" that gives the impression data is served, when the real screen either gets no data or is actually fed by a different mechanism). See `templates/integration-mirage-mock.md`.
9. A MirageJS route with a hardcoded response and no `window.__<feature>MockConfig` failure branch — makes the error UI for that endpoint untestable without editing the route file itself.
10. An ad-hoc `page.route` registered for an endpoint a feature's MirageJS `routes/index.js` already serves — the two compete/race depending on registration order instead of one clear source of truth.
11. A standalone helper module that wraps `addMirage`/`setMirageMockConfig` (e.g. `pages/helpers/<Thing>.helper.js` exporting `mockX(page)` + a `DEFAULT_X` array). Every Mirage-enabled feature calls `addMirage` directly inside a POM method and exposes `setMockConfig()`; default seed data lives in `tests/specs/<Feature>/fixtures/index.js`. See "Playwright-side layout for Mirage" below.
12. Top-level helper functions or constants shared across specs declared inside a spec file — see "Spec holds the flow and the assertions".

## Decision rule: unit vs. integration
- **Pure logic** (form validation, data normalization, calculation) → extract to an exported function + direct unit test, no `mount`. → `templates/unit-pure-function.md`
- **API/service call** → **never write a unit test for a service module.** Services are already exercised by the integration tests (Playwright hits the real service code through the mocked endpoints), so a service unit test only duplicates that coverage. This applies to every service — including ones with internal pagination loops or payload mapping; cover those behaviors through the integration flow that consumes them. When auditing a PR, flag new service unit tests as unnecessary instead of asking for more of them.
- **Global state** (Vuex/Pinia mutation/action/getter) → unit test the store **only when it contains real logic**: conditional branches (e.g. commit only on success, early return, error feedback dispatch), data transformation/merge (pagination append vs. replace, normalization, filtering, derived getters), or chaining of multiple dispatches/services. A store that only passes data through — a mutation that just assigns `state.x = payload`, an action that just calls the service and commits the result with no branching, a getter that just returns `state.x` — gets **no unit test**; it's already covered by the integration flow. Test only the specific mutations/actions/getters that carry logic, not the whole module for completeness. → `templates/unit-store-vuex.md`
- **Isolated component behavior** (emit an event, computed, show/hide based on prop/state, method wiring):
  - no template involved → test directly via the component's `methods`/`computed`. → `templates/unit-container-no-mount.md`
  - template involved (slot, conditional class, prop passed to a child) → `shallowMount` + `@vue/test-utils`. → `templates/unit-container-shallow-mount.md`
- **Full flow** (real navigation, multiple chained API mocks, multi-step modal, real data rendered from mocks) → Playwright integration via Page Object. → `templates/integration-page-object.md` + `templates/integration-playwright-spec.md`
  - If the feature already has `src/<Feature>/mocks/` built on `miragejs` (check: `grep -r miragejs src/<Feature>` and a `models/`/`factories/`/`routes/` subfolder structure), mock its REST endpoints through that server instead of ad-hoc `page.route` — see "Mocking backend for integration tests" below and `templates/integration-mirage-mock.md`. If the feature has no such folder, stay with the plain `page.route`/`mockFromList` pattern; don't introduce Mirage plumbing for a single spec.

### Before removing an integration test in favor of a new unit test — checklist
It's not enough that the unit test covers "the same function"; confirm the integration test doesn't verify something EXTRA:
- The integration test verifies a real computed value (item count, text formatted from specific business logic) → keep the integration test, even with a similar unit test. Example: a test that "opens the quadrant modal and shows N rows with the right employees" covers more than a unit test that only checks whether the modal opens/closes (the unit test stubs the real component and doesn't validate that the correct data reaches it).
- The integration test only reconfirms that a form validation message appears on screen and the pure function already returns exactly that message (standard `CFormBuilder`/`vee-validate` wiring, e.g. `schema.field.validation.custom`) → safe to remove, it's generic, low-risk wiring, replicated across dozens of other forms.
- Practical rule: if the unit test stubs (`stubs`) the component the integration test would actually exercise, the integration test still covers something the unit test doesn't.

## Before touching a feature's specs: check who "owns" the code
Not all logic that appears in SPA screens lives in the app itself. Components coming from a shared component-library submodule namespace (e.g., `@convenia/performance-organisms` inside `.@convenia/`) belong to the library, not to the consuming app. Before "extracting pure logic into a unit test", run `grep -rl <variable-or-computed-name> .@convenia/src/` — if the logic already lives there, it should already have (or needs) a unit test in the library itself; recreating it in the SPA duplicates work in the wrong place. In that case the SPA's integration test remains legitimate: it tests that the integration (the app's store/wiring) feeds the library component correctly, not the library's internal math.

## Test templates per layer

Each file in `templates/` includes: when to use it, the generic code template (swap the names for your feature), and mutation-proofing/gotcha notes specific to that layer.

| Layer | Template | When to use |
|---|---|---|
| Pure function | `templates/unit-pure-function.md` | validation, calculation, formatting — no mock, no mount |
| Store/Vuex | `templates/unit-store-vuex.md` | only mutations/actions/getters with real logic (branching, transformation, chaining), services mocked — pass-through store code is not tested |
| Options API container, no mount | `templates/unit-container-no-mount.md` | `methods`/`computed` with no template logic |
| Container/component via `shallowMount` | `templates/unit-container-shallow-mount.md` | behavior tied to the rendered template |
| Page Object Model | `templates/integration-page-object.md` | API mocks reused across specs for a screen |
| Playwright spec | `templates/integration-playwright-spec.md` | full flow consuming the Page Object |
| Helper shared between specs | `templates/integration-shared-helper.md` | 2+ specs repeat the same post-setup interaction |
| REST mock server (MirageJS) | `templates/integration-mirage-mock.md` | feature already ships `src/<Feature>/mocks/` on `miragejs` — models/factories/routes plus the Playwright-side `addMirage`/`setMirageMockConfig` bridge |
| Anti-patterns | `templates/anti-patterns.md` | catalog of common mistakes to review/avoid |
| Field notes (living log) | `templates/field-notes.md` | lessons learned across sessions — read before starting, append when done, see "Continuous learning loop" |

## Mocking backend for integration tests: MirageJS vs plain `page.route`

Convenia's admin SPA has two coexisting integration-mock mechanisms — pick per
feature, not per spec:

- **MirageJS** (`src/<Feature>/mocks/{models,factories,routes,seeds,fixtures}`,
  aggregated centrally in `src/mocks/*`, bootstrapped in `src/main.js` behind
  `VITE_USE_MOCK`): an in-browser fake REST server with real status codes,
  pagination, and a `schema.db`. Currently used by `Chat`, `Files`,
  `Specialist`, `Disc` (fully REST), `Settings` (SystemFields domain),
  `Recruitment` (JobPosition tags only) and `Strategic` (REST download
  endpoint only — its report data is GraphQL, mocked separately). Before
  writing, grep `src/mocks/bootstrap/index.js` for the current list. Playwright seeds it
  pre-navigation via `addMirage(page, { storageKey, factories, mockConfig })`
  and flips runtime failure scenarios via
  `setMirageMockConfig(page, { storageKey, mockConfig })`, both from
  `tests/utils/index.js`. Full pattern: `templates/integration-mirage-mock.md`.
- **Plain `page.route` / `mockFromList`** (`templates/integration-page-object.md`):
  every other feature. GraphQL calls, regardless of the feature, are mocked
  with `graphQLMocker.setQuery(...)`/`setQueryWithErrors(...)` (dynamic
  builder-based response) or `mockFromList(fixtureJson, page, graphQLMocker)`
  (a recorded request/response snapshot matched by query name + variables) —
  never through Mirage.

**Decision rule when asked to write a new integration test:**
1. Detect the feature's existing mechanism first (`grep -r miragejs
   src/<Feature>`) — never introduce Mirage into a feature that doesn't
   already have it, and never add a competing `page.route` for an endpoint a
   Mirage route already serves (double-mocking causes route-order races).
2. Within a Mirage-enabled feature, still check whether the specific endpoint
   being tested is REST (→ add/extend a Mirage route) or GraphQL (→
   `graphQLMocker`) — a feature can legitimately use both for different
   endpoints, as `Strategic` does.
3. A brand-new feature with a pure-REST backend may adopt Mirage from scratch
   only as a deliberate call (consistency with sibling AI/product features),
   not as the silent default — flag it as an open decision rather than
   assuming it.

## Migrating a feature's mocks to MirageJS (delegate to `convenia-mirage-mocks`)

Trigger: the task asks to migrate a spec/feature from `page.route`/`mockFromList`
REST fixtures to MirageJS, or to add Mirage routes/models/factories to a
feature (including extending an existing Mirage feature with a new domain or
endpoint).

1. **Invoke the peer skill `convenia-mirage-mocks:mirage-mocks`** (Skill tool)
   for everything under `src/`: `src/<Feature>/mocks/{models,factories,routes,fixtures}`,
   the central aggregators in `src/mocks/{models,factories,routes,seeds,fixtures}/index.js`,
   the `applyMockBootstrap(server, '<key>')` line, and the path helpers in
   `src/<Feature>/services/**/names.js` that both the service and the Mirage
   route import. Feed it the endpoint list from the feature's services and
   the JSON fixtures being replaced (status codes, payload shapes, error
   messages) as the API contract.
   - If the skill isn't available (plugin `convenia-mirage-mocks` not
     installed or not loaded), **say so explicitly** and suggest
     `/plugin install convenia-mirage-mocks@convenia-ai-devtools`; then fall
     back to `templates/integration-mirage-mock.md`. Never silently skip it.
2. **This skill keeps the test side**: decide what to migrate (REST only —
   GraphQL stays on `graphQLMocker`/`mockFromList`), wire the POM, move the
   seed data to fixtures, rewrite the spec, delete the replaced JSON
   fixtures, and audit with the anti-patterns above (dead mock, route with no
   failure flag, competing `page.route`, standalone Mirage helper).
3. **Check against what's already in the repo**, not only the templates:
   open at least one sibling Mirage feature (`Settings/SystemFields` for a
   domain-split feature, `Files`/`Disc` for a flat one) and the repo docs
   `mocks_with_mirage.md` / `tests_with_mirage.md` when present, and match
   their layout.

### Playwright-side layout for Mirage (match exactly)
- **`addMirage` goes directly in a POM method** (`mockAPIData`, `seed<Things>`,
  `mock<Things>`), with the defaults as parameters
  (`{ things = defaultThings, mockConfig = {} } = {}`). No standalone helper
  module wrapping it, even when two POMs seed the same data — repeat the
  ~8-line `addMirage` call in each POM.
- **Runtime failure flags go through a POM method named `setMockConfig`**
  that calls `setMirageMockConfig(this.page, { storageKey, mockConfig })`.
- **Default seed data goes in `tests/specs/<Feature>/fixtures/index.js`**,
  exported as `default<Things>`. When two sub-features (`<Feature>/A`,
  `<Feature>/B`) share it, put it at the feature root fixtures, never
  import from inside a sibling sub-feature's `pages/`.
- **`storageKey` is per mock domain**: a flat feature uses the feature key
  (`'files'`, `'disc'`); a feature whose mocks are split by domain
  subfolder (`src/<Feature>/mocks/routes/<Domain>/`) uses `<feature><Domain>`
  (`'settingsSystemFields'`, `'specialistChat'`). The same key goes in
  `applyMockBootstrap`, in the route's `window.__<key>MockConfig`, and in
  every POM `addMirage`/`setMirageMockConfig` call.
- **One factory file per model** (`factories/[<Domain>/]<model>Factory.js`),
  with `index.js` only spreading them.

## Mock isolation between tests (Vitest)
`mockResolvedValueOnce`/`mockRejectedValueOnce` queue values on the mock for the **whole module**, not per `it()`. If a `describe` has more than one function/scenario using the same mock (e.g., `rest.get` called by several service functions, or by several tests of the same function), a value queued by a previous test that wasn't consumed "leaks" into the next test — classic symptom: a call count or return value that doesn't make sense looking only at the failing test, because the cause is in the PREVIOUS test. Rule: whenever a unit test file has more than one `it()` using the same module mock, add `beforeEach(() => { vi.resetAllMocks() })` in the top-level `describe` — not `clearAllMocks` (which only clears call history, not the `mockResolvedValueOnce` queue).

## Spec holds the flow and the assertions; interactions and shared constants live in the Page Object
A spec file declares the `test.extend` fixture, the `describe`/`beforeEach`, and the tests' steps and `expect`s. Nothing else is defined at the top of the spec:
- **No top-level interaction functions in a spec** (`const navigateToX = ...`, `const openYAction = async ({ page }) => ...`, `const closeFeedback = ...`). Each becomes a POM method that uses `this.page` (e.g. `navigateToNextStages()`, `openTagAction({ tagId, action })`, `closeFeedback({ testId })`), and the spec calls `pom.method(...)`. Navigation shortcuts wrap the existing POM navigation method instead of duplicating it.
- **Constants used by 2+ specs, or by a spec and its POM** (entity ids, card/row testids, fixed URLs) go in `tests/specs/<Feature>/<Area>/pages/constants.js`, imported by both the POM and the specs — one source of truth. Reference in the repo: `tests/specs/Benefits/Details/BulkEdit/pages/constants.js`. Export the raw id and derive testids from it (`` `candidate-card-${CANDIDATE_ID}` ``) so other testid shapes (`selectable-card-`, `row-`) reuse the same id.
- **A constant used only in one spec stays in that spec.** A value that already exists in a fixture (e.g. a seeded record's id) is derived from it (`const [ { id: EDITABLE_TAG_ID } ] = defaultTags`), never re-typed.
- **When you move a constant to `pages/constants.js`, replace the same literal in the sibling specs/POMs of that area too** (testid strings, `navigateTo({ id })` args, glob URL strings via template literal). Leave literal regexes and recorded JSON fixtures alone.

## Integração: extracting a helper shared between specs (Playwright)
New shared interactions go on the Page Object (previous section). `tests/specs/<Feature>/specs/helpers/` modules exist in some features; keep using them where they already exist, but don't add new spec-level helper modules when a POM method fits. When 2+ specs repeat the **same post-setup interaction sequence** (not the setup itself, which can legitimately differ per entry point), extract a helper function to `tests/specs/<Feature>/specs/helpers/<name>.js`, keeping each spec's specific setup in the original file. Rule: only extract the part that is identical across the specs; if the setup differs (different route, different mock), don't force both into a single helper — the helper covers only the real intersection, the rest stays at the call site. Full template in `templates/integration-shared-helper.md`.

## Integration rules (Playwright)
- Selectors: always `getByTestId`/`getByRole`. Avoid a loose CSS selector when a testid exists.
- Never `page.waitForTimeout()`/manual `setTimeout`. Use `expect(locator)` with auto-wait (`expect.timeout` is already configured in the project's `playwright.config.js` — usually 5s local / 10s CI-container).
- Page Object Model: reuse existing methods (`mockAPIData`, `start`, `mockPageSetup`) instead of duplicating mocks in each spec. If several specs of the same feature repeat the same mock block in `beforeEach`, add a convenience method to the Page Object (e.g., an `mockXPageSetup()` that aggregates the base `mockPageSetup()` plus sub-area-specific mocks) — see `templates/integration-page-object.md`.
- **Default every new mock to a JSON fixture + `mockFromList`** (see `templates/integration-page-object.md`), not a hand-written `page.route(...).fulfill(...)` — this is the single most repeated PR review comment across features, so treat it as the starting point, not something to fix after review flags it. This includes POST/PUT create/edit mocks: even though the body comes from a form, a Playwright test fills that form with fixed literal values (e.g. `fillTemplateForm({ name: 'Novo template de teste', ... })`), so the real request body is fully deterministic — trace it once (component → payload-building logic → service call) and hardcode the resulting shape as the fixture's `postData`. Don't assume "the body comes from a form" makes it unmockable; only a body that genuinely varies between runs (a generated id/timestamp, per-test-parametrized input) does.
  - When the fixture's route overrides an endpoint another mock already serves on the same path (e.g. a list's base success `GET` mock, with a `POST` create mock layered on top), **don't register a second, narrower `mockFromList` call** — it replaces the whole route handler and the earlier method's mock stops firing, since `mockFromList`'s handler never falls back to a previously-registered one. Instead spread the base fixture's array together with the new entry into ONE call: `mockFromList([ ...GetItems, ...CreateItemSuccess ], this.page, this.graphQLMocker)`.
  - **JSON fixtures live in `tests/specs/<Feature>/data/`** (subfolders by domain allowed, e.g. `data/Goals/`, `data/Objectives/`), never in `RequestMocks/`, `mocks/`, `fixtures/` or similar — this is the front-end playbook standard (`convenia/obsidian-front` → `Playbook/testing.md`) and a recurring PR review comment. When creating a new feature's fixtures, start in `data/`; when touching a feature that still uses `RequestMocks/`, rename the folder with `git mv` and update every import path (`grep -rl "<Feature>/RequestMocks"`), then confirm with eslint and `npx playwright test tests/specs/<Feature> --list`. Don't confuse this with `src/<Feature>/mocks/` (MirageJS server), which keeps its own name.
  - The only case that still needs a hand-written, parametrized `page.route` (kept as a named Page Object method, never inlined in the spec) is a list/search endpoint whose response depends on a URL query string — `mockFromList` never inspects query params, so there's no fixture shape that can match it. When that method overrides a route another mock already serves for a different method, keep a `route.fallback()` branch for the other methods.
- One spec file per functional area, not per whole page. When a file passes ~300-400 lines or gets slow, it's a candidate for a split. Name files by area (`<Feature><Area>.spec.js`), preserving the imports/constants each new file actually uses (run lint afterward — see "Commands"). After a split, confirm the total test count didn't change (`npx playwright test <path> --list | tail -1`) before and after.
- When moving/renaming specs, check the file's imports: not every spec uses the absolute alias (`@tests/specs/<Feature>/...`) — some still import mocks via a relative path (`../mocks/X`), which silently breaks when moved to a different folder (`import/no-unresolved` in eslint catches this, but only if you run it). Prefer the absolute alias when fixing, it's more robust to future moves.

## Organizing specs into subfolders (large features)
When a feature accumulates many specs covering more than one clearly distinct screen/route (rule of thumb: more than ~10 specs, or more than one screen/main container), split `tests/specs/<Feature>/specs/` into subfolders by screen domain — don't leave everything loose in a single `specs/`. Illustrative example: a feature with 22 loose specs covering 4 domains (list, single-item detail, form wizard, overview) was reorganized into `specs/{DomainA,DomainB,DomainC,DomainD}/`.

- **Mirror `src` when the container structure is already hierarchical.** E.g.: `containers/Feature/Detail/{SubA,SubB}` in the source code → `specs/Detail/{SubA,SubB}/` in the specs. This makes it trivial to find the test from the component (and vice versa).
- **`pages/`, `mocks/`, `data/` stay single at the feature root** — only `specs/` gets subfolders. Only split those too if the Page Object itself is exclusive to a single domain; otherwise (a Page Object reused by specs from different domains) splitting fragments reused mock methods unnecessarily.
- **The file name keeps the parent domain prefix even inside the subfolder** (e.g., `Cycles/CyclesTemplates.spec.js`, not `Cycles/Templates.spec.js`) — makes it easier to find the right file from an isolated name search (editor, a CI report that only shows the basename), without depending on the full path.
- **Plural/singular convention as a semantic signal**: plural prefix (`Cycles`, `Employees`) for list/collection screens; singular prefix (`Cycle`, `Employee`) for a single instance's detail screen.
- **The top-level `describe()` must be unique within the entire feature** — never duplicate the same text between two files (e.g., two specs for the same screen both with `describe('MyScreen', ...)` become indistinguishable in the Playwright report; use `'MyScreen - Grid View'` / `'MyScreen - List View'`).
- **Every spec in a feature must carry the `{ tag: '@Feature' }` tag** in the top-level describe. When moving/reviewing a feature's files, check with `npx playwright test tests/specs/<Feature> --list --grep-invert "@Feature"` — if it returns any test, some spec is missing the tag.
- After moving files, always run `npx playwright test tests/specs/<Feature> --list` to confirm Playwright's recursive discovery picked up everything at the new paths (no need to touch `playwright.config.js`; `testDir` already discovers `*.spec.js` at any depth).

## Coverage gate: PR diff coverage must clear the minimum threshold
Convenia requires every PR's **diff coverage** (coverage of only the lines the
branch actually changed, not the whole repo) to clear a minimum percentage
before the PR validation passes. The minimum itself is set by the Tech Lead,
not fixed by this skill — today that value is **80%** — and PRs that fall
under a documented exception can bypass the check via the `coverage-bypass`
label instead of being padded with filler tests. Source of truth:
`corpus/convencoes-de-desenvolvimento/front-end/testes/cobertura.md:53-66`
(via the `convenia-playbook:playbook` skill) — re-check it if the number seems
off, don't assume 80% is hardcoded forever.

Run this gate whenever you're asked to validate/prepare a PR for review, or as
the last step of any task that changed application code destined for a PR.

1. **Run the diff-coverage command.**
   ```sh
   yarn test:diff-coverage
   ```
   Requires `python3` + the `diff_cover` pip package, and a coverage run
   already produced (`yarn test:all:coverage`, or `yarn test:unit:coverage` +
   `yarn test:integration:coverage` + `yarn test:merge-coverage-reports` in
   sequence if the combined command fails because some test is red). Compares
   against `origin/main` by default (`--branch` to target another base). The
   percentage prints as `Coverage: NN%` and the full breakdown lands in
   `coverage-reports/complete/diff-coverage.html`.
2. **Check for an exception before writing anything.** If the PR is a ticket-
   linked change, a mass mechanical refactor (e.g., a string rename across
   many files), an emergency hotfix, or a case already negotiated with the
   Tech Lead, say so and point at the `coverage-bypass` label instead of
   forcing coverage up artificially.
3. **≥ threshold → done.** Report the percentage and stop; no filler tests.
4. **< threshold → identify the uncovered new/changed lines** from the HTML
   report (or the `diff-cover` stdout), file by file. Skip files that
   legitimately don't need coverage per the playbook's "Ignorar cobertura"
   list: pure data/id/constant files, thin third-party wrappers, test
   helpers/utilities themselves, or anything dev+reviewer explicitly agreed is
   unnecessary. Also skip service modules (never unit-tested — covered by
   integration) and store code with no logic (pure pass-through
   mutations/actions/getters); if their lines are still uncovered, close the
   gap with the integration flow that consumes them, not a unit test.
5. **For every remaining uncovered file, apply "Decision rule: unit vs.
   integration" above** to pick the right layer (pure function, store with
   logic, container with/without mount, or full Playwright flow), then write
   the test from the matching file in `templates/`.
6. **The new test must still pass the mutation-testing mindset** (see above)
   — a test that only exists to move the coverage number without catching a
   real behavior change is worse than the gap it "fixed": it hides the real
   gap behind a green number. Don't write `toBeTruthy()`/`toHaveBeenCalled()`
   filler just to touch a line.
7. **Re-run `yarn test:diff-coverage`** to confirm the new tests actually
   closed the gap, then report the before/after percentage.

## Review mode: audit branch changes and gate the fixes
A qualitative, interactive companion to the numeric "Coverage gate" above.
Use it when asked to review a branch/PR's tests (e.g. "revise os testes da PR
atual", "liste os pontos de teste dessa branch", `/test-standards review`) —
it flags weak or missing test coverage in the changed lines using the
mutation-testing mindset and the anti-patterns catalog, even when diff
coverage already clears the threshold, then lets the user decide what to do
with each finding instead of silently writing tests (that's "Coverage gate")
or silently doing nothing.

1. **Resolve the diff scope.** Default to `git diff origin/main...HEAD
   --name-only` (or against another base if one was named); fall back to
   staged/unstaged changes if the branch has no tracked upstream yet. Split
   the changed files into source files with logic vs. their test files (same
   directory's `tests/` sibling, or a `.spec.js`/`.test.js` counterpart) — a
   changed source file with no matching test file at all is itself a finding.
2. **For every changed source file, apply the lenses already in this
   skill** — no new criteria, reuse what's documented above:
   - "Mindset: a test only counts if it catches a mutation" checklist.
   - "Common anti-patterns" catalog.
   - "Decision rule: unit vs. integration" (flag a behavior tested at the
     wrong layer too, e.g. a component branch only ever exercised through a
     slow Playwright flow when a `shallowMount` unit test would isolate it
     just as well).
   - "Before touching a feature's specs: check who owns the code" — don't
     flag logic that actually lives in the `.@convenia` submodule.
   - For a MirageJS-enabled feature, also apply "Mocking backend for
     integration tests" and its anti-patterns (dead mock, hardcoded route
     with no failure branch, competing `page.route` for a Mirage-served
     endpoint, standalone Mirage helper) and "Playwright-side layout for
     Mirage" — see `templates/integration-mirage-mock.md`. If the PR is a
     Mirage migration, also invoke `convenia-mirage-mocks:mirage-mocks` to
     check the `src/<Feature>/mocks` side against its conventions.
3. **Write each finding as a short comment**: file:line (or file/behavior
   when no single line applies), what's untested or weak today, and the
   concrete mutation that would slip through unnoticed. Order real gaps (a
   plausible code change nothing catches) before minor/low-risk ones (prop
   pass-through, cosmetic branches) — don't inflate the list with filler to
   look thorough.
   - **Comment format (list and PR comments).** Write in pt-BR and start
     every finding with two tags: `[TIPO DE AJUSTE] [CRITICIDADE]`.
     - The first tag names the **kind of change the author has to make**,
       not the area of the code. Use `[COBERTURA]` for a missing test or an
       assertion too weak to catch a mutation, `[LÓGICA]` for a behavior or
       business-rule change/bug, `[REFATORAÇÃO]` for restructuring with no
       behavior change (duplicated seed, coupling to internals),
       `[PADRONIZAÇÃO]` for not following the team pattern (`resetAllMocks`,
       `vi.hoisted`, POM/fixture layout), and `[REMOÇÃO]` for dead code or an
       unnecessary test. Never use area tags like `[TESTE]`, `[CÓDIGO]` or
       `[COMPORTAMENTO]`.
     - The second tag is `[ALTA]` (a plausible regression nothing catches),
       `[MÉDIA]` (a real gap or risk, but narrower) or `[BAIXA]` (a minor
       assertion, cosmetics, or optional cleanup).
     - Example: `[COBERTURA] [ALTA] O teste começa com a lista vazia e
       confere que continua vazia; se o guard for removido, nada quebra.`
   - When the user picks posting to the PR, publish all approved findings
     as **one** review (`gh api -X POST .../pulls/<N>/reviews` with a
     `comments` array, `event: "COMMENT"`), with each finding inline on its
     line in the diff. Anchor to `side: "LEFT"` when the finding is about
     a removed line.
     - Every code `suggestion` must hold real, runnable code. Trace the
       actual values first (e.g. what a normalizer really returns), and
       never put a placeholder in it.
4. **Halt at a User-Input Gate before touching any file.** This is what makes
   Review mode different from "Coverage gate", which writes the missing tests
   directly once under threshold. Present the findings list, then invoke
   `AskUserQuestion` (per the `user-input-gates` skill's protocol — a real
   tool call, never a numbered list in text) with per-finding or batched
   multi-select options: "Corrigir agora" / "Pular por enquanto" / "Descartar
   (não é um problema real)". Never write a test for a finding the user
   didn't approve.
5. **For every finding approved to fix**, write it with the matching
   `templates/` file for the layer "Decision rule: unit vs. integration"
   picks, then run the affected test file(s) to confirm green before moving
   to the next finding.
6. **Close with a one-line-per-finding outcome** — fixed / skipped /
   discarded — and, if it was also run, the numeric "Coverage gate" result
   alongside it, so the qualitative and quantitative views land together.
7. The "Continuous learning loop" still applies at the end: draft (never
   auto-write) a field note if the review surfaced a genuinely new, reusable
   gotcha, per that section's gate.

## Diagnosis: an integration test is "broken" and I didn't change anything
Before assuming a real regression in the code, rule out environment causes:
- The shared component-library submodule (`.@convenia` or equivalent) is out of date/ahead of the commit the branch expects. Compare `git ls-tree HEAD .@convenia` (expected commit) with `git -C .@convenia rev-parse HEAD` (current commit); fix with `git submodule update .@convenia` — check `git status` inside the submodule first, so you don't discard work in progress there.
- An isolated failure disappears when running the test alone (`-g "test name"`) but shows up when running the whole file in parallel → likely local infrastructure contention (concurrent workers), not an app bug. Run it isolated or with `--workers=1` to confirm before investigating the component. If the failing test changes on each parallel run but disappears with `--workers=1` across repeated runs, it's almost certainly contention, not a regression — don't spend time investigating the code.

## Commands
- `yarn test:unit` / `yarn test:unit run` — unit tests (Vitest).
- `yarn test:integration` — integration tests (Playwright); use `-g "name"` to run a specific test.
- `npx playwright test <file-or-folder> --workers=1` — reproduces/rules out contention flakiness before investigating a real regression.
- `npx playwright test <file-or-folder> --list` — confirms test count and file discovery after moving/splitting specs.
- `yarn test:all:coverage` — full coverage.
- `yarn test:diff-coverage` — branch-diff-only coverage.
- `yarn eslint <files>` — always run after creating/moving specs (catches unused imports when splitting files).
