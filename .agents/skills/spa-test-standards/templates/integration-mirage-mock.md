# Template: REST mock server (MirageJS)

**When to use:** the feature already has (or, for a brand-new feature, its backend is pure REST and a sibling feature in the same app already sets this precedent — check before introducing it) a `src/<Feature>/mocks/` folder built on `miragejs`. Don't bolt this pattern onto a feature that mocks purely via `page.route`/`mockFromList`, and don't add ad-hoc `page.route` handlers for an endpoint a Mirage route already serves — pick one mechanism per endpoint.

**How to tell a feature uses this pattern**: `ls src/<Feature>/mocks/` exists with `models/`, `factories/`, `routes/` subfolders, and `grep -r miragejs src/<Feature>` returns hits. If instead the feature's Playwright specs mock everything via `page.route(...)`/`mockFromList(json, page, graphQLMocker)` with no `src/<Feature>/mocks/miragejs` folder, that feature uses the plain `page.route` pattern — stay consistent with it (`templates/integration-page-object.md`), don't introduce Mirage there without a deliberate decision.

**Scaffolding the src side:** invoke the `convenia-mirage-mocks:mirage-mocks` skill for models/factories/routes/aggregators (see `SKILL.md` § "Migrating a feature's mocks to MirageJS"). The snippets below are the fallback when that skill isn't available, and the reference for the Playwright side, which this skill owns.

**Mirage only covers REST.** If the feature's services call GraphQL (check `src/<Feature>/services/index.js` for a `middleware(query.X, ...)` call vs. a REST `fetch`/`axios`), those calls are mocked separately with `graphQLMocker.setQuery(...)` or `mockFromList` — never assume every network call in a Mirage-enabled feature goes through the mock server. A feature can legitimately mix both (Mirage for its REST endpoints, `graphQLMocker` for its GraphQL ones).

## Feature-local mock server pieces

```js
// src/MyFeature/mocks/models/index.js
import { Model } from 'miragejs'

export default {
  item: Model,
}
```

```js
// src/MyFeature/mocks/factories/itemFactory.js
import { Factory } from 'miragejs'
import { faker } from '@faker-js/faker'

export default {
  item: Factory.extend({
    id () { return faker.string.uuid() },
    name () { return faker.commerce.productName() },
    created_at () { return '2026-02-28T14:00:00+00:00' },
  })
}
```

```js
// src/MyFeature/mocks/routes/index.js
import { Response } from 'miragejs'
import { get_items, delete_item } from '../../services/names'
import { paginate } from '../../../mocks/utils'

// window.__<feature>MockConfig is how a Playwright spec flips this route to
// a failure scenario at runtime without redefining the handler — see
// `setMirageMockConfig` below.
const getMockConfig = () => window.__myFeatureMockConfig || {}

export default function myFeatureRoutes (restPrefix) {
  this.get(`${restPrefix}${get_items()}`, (schema, request) => {
    if (getMockConfig().failItemsList) {
      return new Response(500, {}, { message: 'Houve um erro ao listar os itens.' })
    }

    const { per_page = 20, cursor = 1 } = request.queryParams
    const { paginatedItems, meta } = paginate(schema.db.items, cursor, per_page)

    return new Response(200, {}, { data: paginatedItems, meta })
  })

  this.delete(`${restPrefix}${delete_item()}`, (schema, request) => {
    const { itemId } = request.params || {}
    if (!schema.db.items.find(itemId)) {
      return new Response(404, {}, { message: 'Item nao encontrado.' })
    }
    schema.db.items.remove(itemId)
    return new Response(204, {}, '')
  })
}
```

```js
// src/MyFeature/mocks/seeds/index.js
export default function myFeatureSeeds (server) {
  server.createList('item', 10)
}
```

**Wire the feature into the central aggregators** (`src/mocks/{models,factories,routes,seeds}/index.js`) by spreading/calling the feature's module — one line added per aggregator, matching the existing entries for the other features. Forgetting this step means the model/factory/route exists but the shared `makeServer()` never sees it (a "dead mock" — see anti-patterns below).

## Playwright side: seeding and runtime config

```js
// tests/specs/MyFeature/fixtures/index.js
export const defaultItems = [
  { id: 'item-1', name: 'Item 01' },
]
```

```js
// pages/MyFeaturePage.js
import { addMirage, setMirageMockConfig } from '@tests:utils/index.js'
import { defaultItems } from '../fixtures'

export default class MyFeaturePage {
  constructor (logged) {
    this.logged = logged
    this.page = logged.page
  }

  async mockAPIData ({ items = defaultItems, mockConfig = {}, createRandom = 0 } = {}) {
    await this.logged.mockAPIData()
    await addMirage(this.page, {
      storageKey: 'myFeature',
      factories: {
        items: { factoryKey: 'item', createRandom, values: items }
      },
      mockConfig
    })
    await this.page.goto('/my-feature')
  }

  // Flip a mockConfig flag AFTER the page already navigated/mocked, to
  // simulate a failure on a subsequent action without re-mocking everything.
  async setMockConfig (mockConfig = {}) {
    await setMirageMockConfig(this.page, { storageKey: 'myFeature', mockConfig })
  }
}
```

```js
// specs/MyFeature.spec.js
test('should show an error when the items list fails', { tag: '@always' }, async ({ myFeaturePage, page }) => {
  await myFeaturePage.mockAPIData({ mockConfig: { failItemsList: true } })

  await expect(page.getByText('Houve um erro ao listar os itens.')).toBeVisible()
})
```

`addMirage` writes `{ factories, mockConfig }` to `window.sessionStorage` under `__<storageKey>_mock_bootstrap__` via `page.addInitScript` — it must run **before** `page.goto`, since the app reads it once at boot (`src/main.js` → `applyMocksBootstrap`) to seed the in-browser Mirage `Server`. `setMirageMockConfig` updates the same storage key plus `window.__<storageKey>MockConfig` directly (no reload needed), which is what a route handler's `getMockConfig()` reads on the next request.

## Rules

- **`storageKey` must match exactly** the key name `applyMocksBootstrap` (`src/mocks/bootstrap/index.js`) uses for that feature (e.g. `'chat'`, `'files'`, `'strategic'`) — a typo silently no-ops (the bootstrap function returns early on a missing/unparseable `sessionStorage` entry) instead of failing loudly. For a feature whose mocks are split by domain subfolder (`src/<Feature>/mocks/routes/<Domain>/`), the key is `<feature><Domain>` (e.g. `'settingsSystemFields'`), and the route reads `window.__<feature><Domain>MockConfig`.
- **`addMirage` lives in the POM method, never in a standalone helper module**; failure flags go through a POM method named `setMockConfig`; default seed data lives in `tests/specs/<Feature>/fixtures/index.js` as `default<Things>` (at the feature root when sibling sub-features share it).
- **One factory file per model** (`factories/[<Domain>/]<model>Factory.js`), `index.js` only spreads them.
- **`factoryKey` must match a real key exported from `src/<Feature>/mocks/factories/index.js`** (e.g. `conversation`, not `conversations`) — `server.createList(factoryKey, ...)` throws at runtime with an unregistered key, but a silently-wrong-but-registered key (e.g. reusing another feature's factory) creates the wrong shape of data with no error at all.
- **Every failure branch a route can return must be reachable from a `mockConfig` flag**, not hardcoded — if a route always returns success, no spec can exercise the error UI without editing the route file itself.
- **Prefer `values` (explicit records) over `createRandom` (faker-generated)** when the spec's assertions depend on specific field values (an id referenced later, a name asserted on screen); reserve `createRandom` for scenarios that only need "N items exist" (pagination, list-length checks).
- **`schema.db` mutations in a route are NOT reset between tests in the same worker** unless the app creates a fresh `Server` per page load (it does, via `main.js` on every `page.goto`/reload) — don't rely on state persisting across a `page.reload()` inside the same test unless that's the intended scenario.
