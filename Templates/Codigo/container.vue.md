---
tags:
  - template
  - frontend
  - vue
date: 2026-09-24
---

# Template: container.vue

Esqueleto do container fino de card. Guias: [[Playbook/container]], [[Playbook/component-conventions]] (ordenacao de atributos). Tokens: [[Playbook/visao-profile]].

```vue
<script setup>
import { computed, ref, provide } from 'vue'
import * as types from '<ROOT_ALIAS>types'
import { useRoute } from 'vue-composition-wrapper'
import { mapState, mapActions } from '@convenia/macros/vuex-composition.macro'
import <Entity>Card from '<ORGANISM_IMPORT>/<Card>.vue'

const route = useRoute()

const { <canEditState> } = mapState('<canEditModule>')
const { <entity>, fieldsMetadata } = mapState('<STORE_MODULE>')

const {
  [types.<TYPE_PREFIX>_GET_<AREA>_<ENTITY>]: get<Entity>,
  [types.<TYPE_PREFIX>_DELETE_<AREA>_<ENTITY>]: delete<Entity>,
  [types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>]: update<Entity>,
  [types.<TYPE_PREFIX>_UPDATE_<AREA>_<ENTITY>_FILES]: update<Entity>Files, // so cards com attachment
  [types.<TYPE_PREFIX>_OPTIONS_GET_<DEPENDENT_OPTION>]: getDependentOptionAction, // so cards com select dependente
} = mapActions()

const canEdit = computed(() => <CAN_EDIT_EXPR>)
const isLoading = ref(false)

// so cards com select dependente — omitir quando nao houver
const getDependentOption = async (parentValue) => {
  isLoading.value = true
  const [ , data ] = await getDependentOptionAction(parentValue)
  isLoading.value = false
  return data
}
provide('getDependentOption', getDependentOption)

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
      :<entity>="<entity>"
      :fields-metadata="fieldsMetadata?.<sub>"
      :is-loading="isLoading"
      @remove="onRemove"
      @submit="onSubmit"
    />
  </div>
</template>
```

Atributos ordenados: `can-edit`, `<entity>`, `fields-metadata`, `is-loading` (alfabetico, sem estrutural aqui) → `@remove`, `@submit` (eventos, alfabetico, por ultimo).

> **Exemplo real:** em cards System+Custom Fields o metadado chega em dois eixos e o container passa duas props (`:system-fields="systemFields?.<sub>" :custom-fields="customFields?.<sub>"`) em vez de uma `:fields-metadata`. Mantendo a ordem alfabetica: `can-edit`, `custom-fields`, `<entity>`, `is-loading`, `system-fields` → `@remove`, `@submit`.

Cards com attachment adicionam props de capacidade derivadas do metadado (nomes definidos pelo organism, confirmar no fragment antes de fixar):

```js
const canRemoveFiles = computed(() => !fieldsMetadata.value?.<sub>?.attachment?.mandatory)
const canViewAttachments = computed(() => !!fieldsMetadata.value?.<sub>?.attachment)
```

## Referencias cruzadas

- [[Playbook/container]]
- [[Playbook/component-conventions]]
- [[Playbook/visao-profile]]
