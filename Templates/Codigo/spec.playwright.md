---
tags:
  - template
  - frontend
  - testing
  - playwright
date: 2026-09-24
---

# Template: spec.playwright

Set canonico de teste Playwright pra um card single-item. Guia: [[Playbook/testing]]. Tokens: [[Playbook/visao-profile]].

```js
import { expect, mockDefault<Group>, <group>Tags, test } from './setup'

test.describe('<Card>', <group>Tags, () => {
  test.describe.configure({ mode: 'serial' })
  test.beforeEach(mockDefault<Group>)

  // 1. @always — renderiza card + abre modal, valida SYSTEM fields
  test('should render the card and open the edit modal with the system fields', { tag: '@always' }, async ({ page, <group> }) => {
    await <group>.goTo<Group>Page()
    const card = page.locator('<CARD_CLASS>')
    await expect(card).toBeVisible()
    await card.getByTestId('info-edit-button').first().click()
    await expect(page.locator('.c-form-builder-modal')).toBeVisible()
  })

  // 2. custom + system em CARD e MODAL — um custom field por tipo (string/list/multiple/boolean/date)
  test('should render all custom and system field values in the info card and edit modal', async ({ page, <group> }) => {
    await <group>.mockGetEmployeeFieldsWithCustomFields()
    await <group>.mockGetEmployee<Card>WithCustomFields()
    await <group>.goTo<Group>Page()
    // ...uma asserção por tipo, no card e no modal — reusar mocks de Storybook do organism
  })

  // 3. descarta ao cancelar
  test('should discard changes when canceling the edit modal', async ({ page, <group> }) => {
    await <group>.goTo<Group>Page()
    await page.getByTestId('info-edit-button').first().click()
    await page.getByTestId('input-abstract-<name>').fill('<changed>')
    await page.getByTestId('form-builder-modal-cancel-button').click()
    await expect(page.getByTestId('form-builder-modal-submit-button')).toBeHidden()
  })

  // 4. atualiza SYSTEM fields
  test('should update system fields successfully', async ({ page, <group>, customExpectations }) => {
    await <group>.mockUpdateEmployee<Card>()
    await <group>.goTo<Group>Page()
    await page.getByTestId('info-edit-button').first().click()
    await page.getByTestId('form-builder-modal-submit-button').click()
    await customExpectations.hasFeedback({ page, type: 'success', message: '<success msg>', index: 0 })
  })

  // 5. atualiza CUSTOM fields — reabre e valida persistencia
  test('should update custom fields successfully', async ({ page, <group>, customExpectations }) => {
    await <group>.mockGetEmployeeFieldsWithCustomFields()
    await <group>.mockGetEmployee<Card>WithCustomFields()
    await <group>.mockUpdateEmployee<Card>WithCustomFields()
    await <group>.goTo<Group>Page()
    // ... editar, submeter, reabrir, validar persistencia
  })

  // 6. erro — feedback de erro + modal mantem estado
  test('should display error feedback and keep the modal state when update fails', async ({ page, <group>, customExpectations }) => {
    await <group>.mockGetEmployeeFieldsWithCustomFields()
    await <group>.mockGetEmployee<Card>WithCustomFields()
    await <group>.mockUpdateEmployee<Card>WithCustomFieldsError()
    await <group>.goTo<Group>Page()
    // ... editar, submeter, validar feedback de erro e que os dados ficam retidos
  })
})
```

Attachment (so cards com anexo, dois testes extras): substituir (POST + DELETE) e remover (DELETE apenas), via `createDocumentFilesRequestTracker` + `expectRequests`.

Fixture de update precisa ser sequencia no mesmo path `[GET inicial, POST exato, GET atualizado]` — corpo do POST capturado, nunca adivinhado.

## Referencias cruzadas

- [[Playbook/testing]]
- [[Playbook/visao-profile]]
