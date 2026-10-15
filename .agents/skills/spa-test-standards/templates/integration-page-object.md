# Template: Page Object Model (Playwright)

**When to use:** every feature with integration specs should have a Page Object per area/screen, reused by all specs for that area — avoids duplicating API mocks in each spec file.

Minimal structure: `mockAPIData` (loads the feature's baseline JSON fixtures via `mockFromList`), `start` (navigates + waits for the page to load), and convenience methods for scenario-specific mocks (success/error variants, sub-area setup) reused by several specs.

## Default: JSON fixture + `mockFromList`

This is the default for every mocked request — GraphQL or REST, success or error, **including POST/PUT create/edit mocks** — unless it hits the one exception below. Record the request/response as a small JSON array (`path`/`queryName`, `method`, `statusCode`, optional `postData`, `responseData`) under `data/`, then load it with the shared `mockFromList` helper (`@convenia/dev-utils/playwright/mock/fromList`, sometimes imported as `mockRequestsFromList` — same function).

A POST/PUT body that comes from a form is not automatically non-deterministic: a Playwright test fills the form with fixed literal values, so the real request body is the same on every run. Trace it once — component → payload-building logic → service call — and hardcode the resulting shape as the fixture's `postData` (`mockFromList` hashes it with `object-hash`, `unorderedObjects: true` by default, so JS key order in the fixture doesn't matter; array order does). Only a body that genuinely varies between runs (a generated id/timestamp, per-test-parametrized input) can't be fixture-matched this way.

```js
// pages/MyFeaturePage.js
import mockFromList from '@convenia/dev-utils/playwright/mock/fromList'

import GetItems from '@tests/specs/MyFeature/data/GetItems.json'
import GetItemsError from '@tests/specs/MyFeature/data/GetItemsError.json'

export default class MyFeaturePage {
  constructor(basePage, graphQLMocker) {
    this.basePage = basePage
    this.page = basePage.page
    this.graphQLMocker = graphQLMocker
  }

  async mockAPIData() {
    mockFromList(GetItems, this.page, this.graphQLMocker)
  }

  // Scenario-specific override: still just a JSON fixture + mockFromList,
  // registered on top of the base mock (last-registered handler wins for that route).
  mockItemsListError() {
    mockFromList(GetItemsError, this.page, this.graphQLMocker)
  }

  async start() {
    await this.page.goto('/my-feature')
    await this.page.getByTestId('my-feature-header').waitFor()
  }
}
```

```json
// data/GetItemsError.json
[
  {
    "path": "**/*/api/my-feature/items",
    "method": "GET",
    "statusCode": 500,
    "responseData": {}
  }
]
```

### Overriding a route another mock already serves (e.g. a POST create mock on the same path as the base GET list mock)

Don't register a second, narrower `mockFromList` call for the same route pattern — it replaces the whole handler, so the earlier method's mock (the base GET) stops firing entirely (`mockFromList`'s handler never falls back to a previously-registered one, unlike a hand-written `page.route` with an explicit `route.fallback()`). Instead spread the base fixture's array together with the new one into a single call:

```js
  // The create POST shares its route with the base GetItems GET mock, so both
  // entries must go through the SAME mockFromList call, not two separate ones.
  mockCreateItem({ status = 200 } = {}) {
    const fixture = status === 200 ? CreateItemSuccess : CreateItemError

    mockFromList([ ...GetItems, ...fixture ], this.page, this.graphQLMocker)
  }
```

```json
// data/CreateItemSuccess.json — postData is the exact body the form always sends in this test
[
  {
    "path": "**/*/api/my-feature/items",
    "method": "POST",
    "statusCode": 200,
    "postData": { "name": "New item from test" },
    "responseData": { "data": { "id": "new-item-id" } }
  }
]
```

## Exception: custom `page.route`, still centralized in the Page Object

Only reach for a hand-written `page.route(...).fulfill(...)` for a list/search endpoint whose response depends on a URL query string — `mockFromList` never inspects query params, so no fixture shape can match it. Keep it as a named method here instead of inlining it in the spec:

```js
  // Exception: the response depends on a URL query string mockFromList can't match.
  mockFilteredItems({ status = 200, query } = {}) {
    const filteredRoute = `**/*/api/my-feature/items?filter=${query}`

    this.page.route(filteredRoute, (route) => {
      if (route.request().method() !== 'GET') return route.fallback()
      if (status !== 200) return route.fulfill({ status })

      return route.fulfill({ status, json: { data: [] } })
    })
  }
```

`route.fallback()` for every other method is what makes this safe to layer on top of a route another mock already serves.

**Rule:** if several specs of the same feature repeat the same mock block in `beforeEach`, add a convenience method to the Page Object (like `mockItemDetailPageSetup` below) instead of duplicating the block in each spec.

```js
  // Convenience: aggregates the base setup + sub-area-specific mocks,
  // reused by all specs in that sub-area instead of repeating the block in every beforeEach.
  async mockItemDetailPageSetup({ item } = {}) {
    await this.mockAPIData()
    mockFromList(item ? [ item ] : GetItemDetail, this.page, this.graphQLMocker)
  }
```
