---
tags:
  - playbook
  - frontend
  - vue
  - conventions
date: 2026-09-24
---

# Convencoes de Componente (Organism/Vue)

## O que e

Regras de autoria de componente Vue que valem pra qualquer organism/componente compartilhado consumido por SPA — nao sao especificas de nenhum card ou dominio. Complementam [[Playbook/container]] (que cobre o container fino da SPA) com o lado do componente que a SPA consome.

## Props e emits em ordem alfabetica ascendente

Declare as chaves de `defineProps`/`defineEmits` em ordem alfabetica ascendente. O mesmo vale pra `argTypes`/`args` de Storybook (ver abaixo). Isso mantem diff estavel e previsivel — quem le o componente sabe onde procurar uma prop sem depender da ordem de escrita de quem editou por ultimo.

```js
defineProps({
  canEdit: Boolean,
  entity: [ Array, Object ],
  fieldsMetadata: Object,
  isLoading: Boolean,
})
```

## Ordem de atributos no template

Ordene os atributos de todo componente assim:

1. Diretivas estruturais primeiro — `v-if`, `v-model`.
2. Todos os demais atributos (estaticos, `:bound`, boolean shorthand) juntos, **em ordem alfabetica pelo nome**, ignorando o prefixo `:`.
3. Handlers `v-on`/`@evento` por ultimo, tambem em ordem alfabetica.

```html
<c-form-builder-modal
  v-if="isEditing"
  v-model="formData"
  :fields-options="fieldsOptions"
  :is-loading="isSubmitting"
  label-left
  :schema="formSchema"
  title="<Titulo>"
  @close="onClose"
  @submit="onSubmit"
/>
```

Isso vale tanto pro organism (no template do fragment) quanto pro container da SPA que o consome — ver [[Templates/Codigo/container.vue]] pro esqueleto ja ordenado.

## Comentarios — proibidos, com duas excecoes

Nao deixe comentario no codigo. As unicas excecoes: uma constraint genuinamente nao-obvia que nao da pra expressar em codigo (necessidade extrema), e bloco JSDoc em helper exportado. Prefira nome claro a comentario explicativo — se o nome precisa de comentario pra fazer sentido, o nome esta errado.

Vale tambem pra Storybook: nenhum arquivo de story leva comentario — os mocks e nomes de variante sao a documentacao.

## Imports — nomeados do barrel raiz, nunca path profundo

Componentes de biblioteca compartilhada se importam pelo barrel raiz do pacote, com import nomeado — nunca por um path interno/profundo:

```js
// certo
import { ConfirmationModal } from '@convenia/common-organisms'

// errado — path profundo
import ConfirmationModal from '@convenia/common-organisms/Modals/Confirmation'
```

O binding importado define a tag no template (`ConfirmationModal` renderiza como `<confirmation-modal>`).

## Consts de enum — sempre `Object.freeze`

Todo enum/const de grupo local e congelado, nunca mutado:

```js
export const <AREA>_SECTIONS = Object.freeze({
  <SECTION>: '<section-key>',
})
```

Declare esses consts num modulo dedicado (`content/consts/<area>.js` ou equivalente), nunca inline num mapper ou componente.

## Naming — substantivo de dominio, nunca generico

Nomeie a prop de entidade de um card com o substantivo do dominio (`personal`, `foreign`, `bankAccounts`) — nunca um nome generico como `values`. O mesmo vale pra qualquer variavel que representa a entidade do card.

## Storybook — autoria (CSF3)

Quando o organism ganha uma story:

- Formato CSF3: export default com `title`, `component`, `tags`, `parameters`, `argTypes`, `args`, e **um unico `render` compartilhado**.
- `argTypes`/`args` em **ordem alfabetica ascendente** — inclusive os `args` no nivel da variante.
- Cada variante e um `export const` que **so passa `args`** — nunca declara o proprio `render`. Se um caso precisa de interacao, drive pelos handlers do `render` compartilhado, nao um render por story.
- Consts auxiliares (stubs de evento, chaves travadas especificas do card) ficam logo **apos os imports**, no topo do arquivo — nunca inline dentro do `export default` ou de uma story.
- Mocks derivados (ex. uma variante "tudo desabilitado") vem de um helper compartilhado — nao reinvente a fabrica de mock por card.

## Referencias cruzadas

- [[Playbook/container]]
- [[Playbook/playbook]]
- [[Templates/Codigo/container.vue]]
