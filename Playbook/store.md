---
tags:
  - playbook
  - frontend
  - vuex
date: 2026-10-01
---

# Store (Vuex)

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. Naming de types fica em [[Playbook/store-types]]. Criação e registro de módulo ficam em [[Playbook/new-module]].

## O que é

A **store** Vuex guarda o estado compartilhado de uma área e é a única camada que chama [[Playbook/services]]. O container despacha uma action; a action chama o service, recebe `[ err, data ]`, faz `commit` de uma mutation e devolve a tupla pro container.

- **Slice**: um módulo Vuex (`{ state, getters, mutations, actions }`) de uma feature.
- **Action**: função async. Chama service, decide feedback, faz `commit`, refaz fetch.
- **Mutation**: função síncrona. Só escreve no `state`.
- **Getter**: valor derivado do `state`.
- **Type**: constante que nomeia action, mutation ou getter (ver [[Playbook/store-types]]).

```
container → action → service → [ err, data ] → commit(mutation) → state → container
```

## Onde fica

```
src/<ROOT>/store/
├── types.js                  # todos os types do módulo — ver Playbook/store-types
└── <Feature>/
    ├── index.js              # slice: { state, getters, mutations, actions }
    └── tests/index.test.js   # teste de erro do slice — ver Playbook/testing
```

- Slice MUST ficar inline num `index.js` só (exceção: `state.js` sibling quando o state fica muito grande).
- Slice MUST NOT usar `namespaced: true`. A unicidade dos nomes vem do valor do type.
- Registro do slice na SPA: ver [[Playbook/new-module]].

## Por que usar

- Um lugar só faz request e trata erro: o container fica fino e não conhece service.
- Dado buscado uma vez fica disponível pra todo container da área.
- Feedback e refetch ficam na action: toda tela que salva o mesmo recurso se comporta igual.

## Como usar

### State

- State MUST guardar só dado reutilizável entre containers (entidade, dado compartilhado entre containers).
- Dado de tela MUST ficar no container (`ref`): modal aberto, form em edição, callback, loading.
- Declare toda chave no state com o valor vazio do tipo dela: `[]` pra lista, `{}` pra objeto, `null` pra valor único. Chave que não está declarada no state não atualiza a tela (o Vue 2 não a observa).
- State MUST NOT começar com dado ou default de negócio. O dado vem do service.
- Chaves MUST ficar em ordem alfabética ascendente.

```js
const state = {
  <entities>: [],
  <entity>: null,
}
```

### Mutations

- Mutation MUST ser síncrona e só escrever no `state`.
- Escrita MUST gerar nova referência e se proteger de `null`/`undefined`: `[ ...(data || []) ]` pra lista, `{ ...(data || {}) }` pra objeto.
- Reset MUST ser uma mutation explícita `RESET_<AREA>`, que volta a chave ao valor vazio.
- Mutations MUST ficar em ordem alfabética pelo nome do type.

```js
const mutations = {
  [types.<TYPE_PREFIX>_<FEATURE>_RESET_<AREAS>]: (state) => {
    state.<entities> = []
  },

  [types.<TYPE_PREFIX>_<FEATURE>_SET_<AREA>]: (state, data) => {
    state.<entity> = { ...(data || {}) }
  },

  [types.<TYPE_PREFIX>_<FEATURE>_SET_<AREAS>]: (state, data) => {
    state.<entities> = [ ...(data || []) ]
  },
}
```

### Actions

#### Signature

- Assinatura padrão: `async ({ commit, dispatch, state }, payload) =>`, com `const { ... } = payload || {}` no corpo. Payload é objeto, nunca posicional.
- Param simples (ex.: `employeeId`, `companyId`) MUST ser destructurado na assinatura: `async ({ commit }, { employeeId } = {}) =>`.
- Quando a assinatura passa do limite de linha do ESLint, use `async (context, payload) =>` e destructure `context` e `payload` nas primeiras linhas do corpo.
- Destructuring de `context` e `payload` no corpo MUST ter proteção `|| {}`: `const { employeeId } = payload || {}`. Default na assinatura (`payload = {}`) só cobre `undefined`; `null` quebra o destructuring.

```js
[types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]: async (context, payload) => {
  const { dispatch, state } = context || {}
  const { employeeId, id, ...data } = payload || {}
  // ...
},
```

#### Regras

- Service MUST ser importado pelo barrel da feature: `import * as services from '<ROOT_ALIAS>:services/<Area>/<Feature>'`.
- Action MUST retornar a tupla completa `[ err, data ]`. `data` é `null` quando não há dado.
- Commit de **dado** só no sucesso. No erro, a action MAY fazer commit de estado de erro ou reset quando a tela depende disso (ex.: limpar a lista, marcar acesso negado).
- Escrita MUST refazer o GET afetado dentro da action (`await dispatch(<GET>)`). O container não refaz fetch.
- Quando o recurso tem create **e** update, os dois MUST ser uma action única `UPDATE_<AREA>`, que decide por `!payload.id`. O container nunca escolhe o branch (ver [[Playbook/container]]). Com só um dos dois, a action chama o service direto.
- Erro com significado de negócio documentado pela API (ex.: `422` = nome duplicado) MAY ter tratamento próprio na action. Sem contrato documentado, qualquer erro segue o fluxo padrão.
- Actions MUST ficar em ordem alfabética pelo nome do type.

```js
const actions = {
  [types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>]: async ({ commit, dispatch }, { employeeId } = {}) => {
    const [ err, data ] = await services.get<Entities>({ employeeId })

    if (!err) commit(types.<TYPE_PREFIX>_<FEATURE>_SET_<AREAS>, data)

    if (err) dispatch('FEEDBACKS_ADD', {
      type: 'error',
      message: '<Entidades>',
      highlighted: 'não foram carregadas.'
    })

    return [ err, data ]
  },

  [types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]: async ({ dispatch }, payload) => {
    const { employeeId, id } = payload || {}
    const isCreating = !id

    const [ err, data ] = isCreating
      ? await services.create<Entity>(payload)
      : await services.update<Entity>(payload)

    if (!err) await dispatch(types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>, { employeeId })

    dispatch('FEEDBACKS_ADD', {
      type: err ? 'error' : 'success',
      message: '<Entidade>',
      highlighted: err ? 'não foi salva.' : 'salva com sucesso.'
    })

    return [ err, data ]
  },
}
```

### Feedback

Feedback é o toast global da SPA. A action dispara `dispatch('FEEDBACKS_ADD', { type, message, highlighted })`.

- `type`: `'success'` ou `'error'`. `message`: texto principal. `highlighted`: complemento em destaque (opcional).
- Leitura MUST disparar feedback só no erro.
- Escrita MUST disparar feedback de sucesso e de erro.
- Action MUST ter no máximo um `dispatch('FEEDBACKS_ADD')` por fluxo. Quando há sucesso e erro, use um único objeto: o texto comum fica fixo e só o que muda usa ternário `err ? <erro> : <sucesso>`.

DO:

```js
dispatch('FEEDBACKS_ADD', {
  type: err ? 'error' : 'success',
  message: '<Entidade>',
  highlighted: err ? 'não foi removida.' : 'removida com sucesso.'
})
```

DON'T:

```js
if (err) dispatch('FEEDBACKS_ADD', { type: 'error', message: '<Entidade> não foi removida' })
else dispatch('FEEDBACKS_ADD', { type: 'success', message: '<Entidade> removida' })
```

### Getters

- Getter MUST ser usado só pra filtragem ou derivação explícita (boolean, lista filtrada, total).
- Pra ler o state, use `mapState` da macro `@convenia/macros/vuex-composition.macro` (ver "Consumo no container").

```js
const getters = {
  [types.<TYPE_PREFIX>_<FEATURE>_HAS_<AREAS>]: ({ <entities> }) => !!<entities>.length,
}
```

### Consumo no container

- Container MUST importar `mapState`, `mapActions`, `mapMutations` e `mapGetters` de `@convenia/macros/vuex-composition.macro`.
- Destructuring com mais de um item MUST ter um item por linha.
- Itens do destructuring MUST ficar em ordem alfabética.

```js
import { mapActions, mapState } from '@convenia/macros/vuex-composition.macro'
import * as types from '<ROOT_ALIAS>:types'

const {
  <entities>,
  <entity>,
} = mapState('<STORE_MODULE>')

const {
  [types.<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>]: get<Entities>,
  [types.<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>]: update<Entity>,
} = mapActions()
```

Quando o container também usa types comuns da SPA, siga a regra de import de [[Playbook/store-types]] (seção "Import"):

```js
import * as types from '@types'
import * as types<Area> from '<ROOT_ALIAS>:types'
```

### Testes

Caminho feliz é coberto pelo E2E Playwright do card. Teste unitário Vitest cobre só fluxo de erro da action, sem duplicar o que o E2E já valida. Regras em [[Playbook/testing]], seção "Teste unitário de store (Vitest)". Esqueleto em [[Templates/Codigo/store.test.js]].

## Anti-patterns

| Anti-pattern | Por que é ruim | Forma certa |
|---|---|---|
| `return [ err ]`, `return err`, sem `return` | container não recebe `data`; contrato quebra | `return [ err, data ]` |
| `return [ false, data ]` | `false` não é "sem erro" no contrato | `[ null, data ]` |
| `try/catch` na action | o service já captura e devolve a tupla; tratamento duplicado | `if (err)` sobre a tupla. Exceção: chamada fora de service que pode lançar erro (ex.: lib de upload), com comentário explicando |
| Action lendo a variável `state` do módulo | só funciona porque state é objeto; quebra fácil | `state` do contexto |
| Commit de dado no erro | tela mostra dado inválido | commit de dado só no sucesso |
| Dois `FEEDBACKS_ADD` no mesmo fluxo | mensagem duplicada; difícil de manter | um objeto com ternário |
| Texto repetido nos dois lados do ternário | duplicação | texto comum fixo, ternário só no que muda |
| Modal, form, callback ou loading no state | dado de tela vaza pra store global | `ref` no container |
| `const { x } = payload` ou `[ ...data ]` sem proteção | `null` vindo do dispatch ou da API quebra a action/mutation | `payload \|\| {}`, `[ ...(data \|\| []) ]`, `{ ...(data \|\| {}) }` |
| State começando com default de negócio | esconde dado faltando da API | valor vazio do tipo |
| Mutation `async` ou chamando callback | Vuex strict reclama; efeito colateral fora da action | lógica async na action |
| `CREATE` + `UPDATE` separados pro mesmo recurso | container escolhe o branch | `UPDATE_<AREA>` único por `!payload.id` |
| Container refaz GET depois de salvar | regra duplicada em cada tela | refetch dentro da action |
| Getter `({ <entities> }) => <entities>` | indireção sem ganho | `mapState` |
| Getter lendo `sessionStorage`/`localStorage` | getter deixa de ser derivado do state | ler no container ou na action |
| `import { mapActions } from 'vuex'` / `this.$store` | foge do padrão Composition | macro `vuex-composition.macro` |
| Destructuring de `mapState`/`mapActions` numa linha só ou fora de ordem | difícil de revisar; diff ruidoso | um item por linha, ordem alfabética |

## Referências cruzadas

- [[Playbook/store-types]]
- [[Playbook/visao-profile]]
- [[Playbook/services]]
- [[Playbook/mappers]]
- [[Playbook/container]]
- [[Playbook/testing]]
- [[Playbook/new-module]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/store-module.js]]
- [[Templates/Codigo/store.test.js]]
