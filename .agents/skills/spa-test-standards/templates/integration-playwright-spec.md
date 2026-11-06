# Template: integration spec (Playwright)

**When to use:** a full flow (real navigation, multiple chained API mocks, multi-step modal, real data rendered from mocks) — consumes the feature's Page Object (see `templates/integration-page-object.md`).

```js
import { test, expect } from '@playwright/test'
import { MyFeaturePage } from '../pages/MyFeaturePage'

test.describe('MyFeature - Item detail', { tag: '@MyFeature' }, () => {
  let myFeaturePage

  test.beforeEach(async ({ page }) => {
    myFeaturePage = new MyFeaturePage(page)
    await myFeaturePage.mockItemDetailPageSetup({ item: { id: 'x', name: 'Test item' } })
    await myFeaturePage.start()
  })

  test('should show the item details after selecting it from the list', async ({ page }) => {
    await page.getByTestId('item-row-x').click()

    await expect(page.getByTestId('item-detail-name')).toHaveText('Test item')
  })
})
```

## Rules

- Selectors: always `getByTestId`/`getByRole`. Avoid a loose CSS selector when a testid exists.
- Never `page.waitForTimeout()`/manual `setTimeout`. Use `expect(locator)` with auto-wait (`expect.timeout` is already configured in the project's `playwright.config.js` — usually 5s local / 10s CI-container).
- Reuse existing Page Object methods (`mockAPIData`, `start`, `mockPageSetup`) instead of duplicating mocks in each spec.
- **Never register a `page.route(...)` directly in a spec file.** A scenario-specific mock (error variant, success variant with a different id) is a Page Object method backed by a JSON fixture + `mockFromList` — see `templates/integration-page-object.md`'s "Default" and "Exception" sections. If a spec needs `page.route`, that's a signal the mock belongs in the Page Object, not that a JSON fixture is the wrong shape for it.
- Every top-level `describe()` carries the `{ tag: '@Feature' }` tag and has a unique text within the entire feature (never duplicate the same text between two files — e.g., two specs covering different views of the same screen use `'Feature - View A'` / `'Feature - View B'`, not `'Feature'` twice).
- One spec file per functional area, not per whole page. When a file passes ~300-400 lines or gets slow, it's a candidate for a split — name files by area (`<Feature><Area>.spec.js`).
