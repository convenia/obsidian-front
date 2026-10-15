# Template: helper shared between specs (Playwright)

**When to use:** 2+ specs repeat the **same post-setup interaction sequence** (not the setup itself, which can legitimately differ per entry point — e.g., the same modal opened from two different screens). Extract a helper function to `tests/specs/<Feature>/specs/helpers/<name>.js`, keeping each spec's specific setup in the original file.

**Rule:** only extract the part that is identical across the specs; if the setup differs (different route, different mock), don't force both into a single helper — the helper covers only the real intersection, the rest stays at the call site.

```js
// specs/helpers/someFlow.js
import { expect } from '@playwright/test'

export const fillAndSubmit = async ({ page, fieldX, fieldY }) => {
  await page.getByTestId('select-fieldX').click()
  await page.getByTestId(`option-fieldX-${fieldX}`).click()
  await page.getByTestId('form-builder-modal-submit-button').click()
}

export const waitForFeedback = async ({ page, customExpectations, modal, type, message }) => {
  if (type === 'success') await expect(modal).toBeHidden()
  else await expect(modal).toBeVisible()
  await customExpectations.hasFeedback({ page, type, message })
}
```

Usage in two different specs that open the same modal from distinct entry points:

```js
// specs/EntryPointA.spec.js
import { fillAndSubmit, waitForFeedback } from './helpers/someFlow'

test('should submit successfully from entry point A', async ({ page }) => {
  // setup specific to EntryPointA
  await fillAndSubmit({ page, fieldX: 'value-x', fieldY: 'value-y' })
  await waitForFeedback({ page, customExpectations, modal, type: 'success', message: 'Saved successfully' })
})
```
