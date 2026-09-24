---
tags:
  - template
  - frontend
  - vue
date: 2026-09-24
---

# Template: container.vue

Esqueleto do container fino de card. Guia: [[Playbook/container]]. Tokens: [[Playbook/visao-profile]].

```vue
<script setup>
import { computed, ref, provide } from 'vue'
import * as types from '<ROOT_ALIAS>types'
import { useRoute } from 'vue-composition-wrapper'
import { mapState, mapActions } from '@convenia/macros/vuex-composition.macro'
import <Entity>Card from '@convenia/employee-organisms/SystemFields/<Group>/fragments/<Card>/<Card>.vue'

const route = useRoute()

// canEdit — fonte muda por visao, ver <CAN_EDIT_EXPR> em Playbook/visao-profile
const { <canEditState> } = mapState('<canEditModule>')
const { <entity>, systemFields, customFields } = mapState('<STORE_MODULE>')

const {
  [types.<TYPE_PREFIX>_GET_<AREA>_<ENTITY>]: get<Entity>,
  [types.<TYPE_PREFIX>_DELETE_<AREA>_<ENTITY>]: delete<Entity>,
  [types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>]: update<Entity>,
  [types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES]: update<Entity>Files, // so cards com attachment
  [types.<TYPE_PREFIX>_OPTIONS_GET_CITIES]: getCitiesAction,               // so cards com select dependente
} = mapActions()

const canEdit = computed(() => <CAN_EDIT_EXPR>)
const isLoading = ref(false)

// so cards com select dependente — omitir quando nao houver
const getCities = async (stateId) => {
  isLoading.value = true
  const [ , data ] = await getCitiesAction(stateId)
  isLoading.value = false
  return data
}
provide('getCities', getCities)

const onRemove = async ({ id, callback }) => {
  const [ err ] = await delete<Entity>({ id, employeeId: route.params.employeeId })
  callback(err)
}

const onSubmit = async ({ id, data, addedFiles, deletedFiles, callback }) => {
  const { employeeId } = route.params

  const [ err, entity ] = await update<Entity>({ ...data, id, employeeId })
  if (err) return callback(err)

  if (addedFiles.length || deletedFiles.length) {
    const [ filesErr ] = await update<Entity>Files({
      employeeId,
      <entity>Id: id || entity?.id,
      files: { upload: addedFiles, remove: deletedFiles.map(fileId => ({ id: fileId })) },
    })
    if (filesErr) return callback(filesErr)
  }

  callback()
  if (!err) await get<Entity>({ employeeId })
}
</script>

<template>
  <div class="<CARD_CLASS>">
    <<entity>-card
      :can-edit="canEdit"
      :is-loading="isLoading"
      :<entity>="<entity>"
      :system-fields="systemFields?.<sub>"
      :custom-fields="customFields?.<sub>"
      @submit="onSubmit"
      @remove="onRemove"
    />
  </div>
</template>
```

Cards com attachment adicionam duas props derivadas do payload:

```js
const canRemoveFiles = computed(() => !systemFields.value?.<sub>?.attachment?.mandatory)
const canViewAttachments = computed(() => !!systemFields.value?.<sub>?.attachment)
```

## Referencias cruzadas

- [[Playbook/container]]
- [[Playbook/visao-profile]]
