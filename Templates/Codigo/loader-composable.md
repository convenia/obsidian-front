---
tags:
  - template
  - frontend
  - loader
date: 2026-09-24
---

# Template: loader-composable

Esqueleto do loader do container pai. Guia: [[Playbook/loader]]. Tokens: [[Playbook/visao-profile]].

```vue
<script setup>
import * as types from '<ROOT_ALIAS>types'
import { provide } from 'vue'
import { useLoaders } from '@convenia/composables'
import { mapState, mapActions } from '@convenia/macros/vuex-composition.macro'
import { getFileUrl } from '@modules/fileHelpers'

import <CardA>Section from './<CardA>'
import <CardB>Section from './<CardB>'

const { addLoader, isLoading } = useLoaders()
const { <entity> } = mapState('<STORE_MODULE>')

const {
  [types.<TYPE_PREFIX>_GET_<AREA>_FIELDS]: getFields,
  [types.<TYPE_PREFIX>_GET_<AREA>_SECTIONS]: getSections,
  [types.COMMON_STATES]: getStates,
  [types.COMMON_OPTIONS_GET_CITIES]: getCities,
} = mapActions()

provide('employeeId', route.params.employeeId || employeeIdFrom<ID_SOURCE>)   // so no pai
provide('getFileUrl', getFileUrl)                                             // quando varios cards compartilham arquivo

addLoader(async () => {
  const { employeeId } = route.params
  await Promise.allSettled([
    getFields({ employeeId }),
    getSections({ employeeId }),
    getStates(),
  ])
  if (<entity>.value?.state?.id) await getCities(<entity>.value.state.id)
})
</script>

<script>
export default { name: '<Module><ContainerArea>Container' }
</script>

<template>
  <div class="<CARD_CLASS>">
    <c-loader v-if="isLoading" class="loader" :size="isMobile ? 79 : 99" />
    <div v-else class="containers">
      <<card-a>-section />
      <<card-b>-section />
    </div>
  </div>
</template>

<style lang="scss">
.<CARD_CLASS> {
  & > .c-loader {
    position: absolute;
    top: 50%;
    left: 50%;
    transform: translate(-50%, -50%);
    margin: 0 auto;
  }
  & > .containers > *:not(:last-child) { margin-bottom: 20px; }
}
</style>
```

`provide` do container filho (`getCities`, escopo daquele card apenas):

```js
const getCities = (stateId) => getStateCities(stateId)   // <TYPE_PREFIX>_OPTIONS_GET_CITIES -> [err, data]
provide('getCities', getCities)
```

Regras a nao esquecer: espacamento/loader ficam so no pai; `provide` de escopo geral (`employeeId`) so no pai, `provide` de escopo de card (`getCities`, `getZipCode`) so no container filho; sem fetch de options pra select que ja vem no payload de fields.

## Referencias cruzadas

- [[Playbook/loader]]
- [[Playbook/visao-profile]]
