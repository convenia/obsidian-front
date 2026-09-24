---
tags:
  - template
  - frontend
  - modulo
date: 2026-09-24
---

# Template: module-scaffold

Esqueleto de arquivos pronto pra copiar ao criar um modulo novo. Guia: [[Playbook/new-module]].

## Arvore minima (modulo standalone, sem menu proprio)

```
src/<Module>/
├── containers/<Feature>.vue
├── services/
│   ├── index.js
│   └── <Feature>.js
├── store/
│   ├── index.js
│   ├── types.js
│   └── <Feature>/index.js
└── routes/
    ├── index.js
    └── names.js
```

## Arvore com secao propria no menu (grouping)

Igual a minima, mais `content/`, `helpers/`, `components/` conforme necessidade, e `routes/index.js` compondo varios arquivos irmãos:

```
src/<Module>/
├── containers/{FeatureA,FeatureB}/...
├── services/<FeatureA>.js  # sem barrel — import direto por path
├── services/<FeatureB>.js
├── store/
│   ├── index.js
│   ├── types.js
│   ├── <FeatureA>/index.js
│   └── <FeatureB>/index.js
├── content/consts/<area>.js
└── routes/
    ├── index.js       # array composto de featureA.js + featureB.js
    ├── names.js
    ├── featureA.js
    └── featureB.js
```

## `jsconfig.json` — entradas a adicionar

```jsonc
"@<Module>/*": ["src/<Module>/*"],
"@<Module>:containers/*": ["src/<Module>/containers/*"],
"@<Module>:services/*": ["src/<Module>/services/*"],
"@<Module>:store/*": ["src/<Module>/store/*"],
"@<Module>:types/*": ["src/<Module>/store/types/*"],
"@<Module>:routes/*": ["src/<Module>/routes/*"]
```

## `store/types.js`

```js
export const <MODULE>_GET_<NOUN> = '<MODULE>/GET_<NOUN>'
export const <MODULE>_SET_<NOUN> = '<MODULE>/SET_<NOUN>'
```

## `store/index.js`

```js
export { default as <slice> } from '@<Module>:store/<Feature>'
```

## `store/<Feature>/index.js`

```js
import * as types from '@<Module>:types'
import * as services from '@<Module>:services/<Feature>'

export default {
  state: {
    <noun>: [],
  },

  getters: {
    [types.<MODULE>_GET_<NOUN>]: ({ <noun> }) => <noun>,
  },

  mutations: {
    [types.<MODULE>_SET_<NOUN>]: (state, data) => {
      state.<noun> = data
    },
  },

  actions: {
    [types.<MODULE>_GET_<NOUN>]: async ({ commit }, params) => {
      const [ err, data ] = await services.get<Noun>(params)
      if (!err) commit(types.<MODULE>_SET_<NOUN>, data)
      return [ err, data ]
    },
  },
}
```

## `routes/names.js`

```js
export const <MODULE> = '<Nome de exibicao>'
export const <MODULE>_<SUBROUTE> = '<Nome de exibicao> - <Subrota>'
```

## `routes/index.js` — variante standalone (um objeto)

```js
import * as names from '@<Module>:routes/names'

export default {
  path: '/<slug>',
  name: names.<MODULE>,
  meta: { title: '<Titulo>', permissionName: '<permission>' },
  component: () => import('@<Module>:containers/<Feature>.vue'),
}
```

## `routes/index.js` — variante array (varias sub-rotas)

```js
import featureA from './featureA'
import featureB from './featureB'

export default [
  featureA,
  featureB,
]
```

## Wiring nos arquivos centrais (diff textual)

`src/store.js`:

```diff
+import * as <module> from '@<Module>:store'

 export default new Vuex.Store({
   modules: {
     ...
+    ...<module>,
   }
 })
```

`src/Common/routes/index.js` — caso standalone:

```diff
+import <module> from '@<Module>:routes'

 const routes = [
   ...
+  <module>,
 ]
```

`src/Common/routes/index.js` — caso com grouping proprio:

```diff
+import <module> from '@<Module>:routes'

 const routes = [
   ...
+  {
+    path: types.GROUPING_<MODULE>_SLUG,
+    meta: { grouping: groupingsList.<Module> },
+    children: <module>,
+    components: {
+      subMenu: () => import('@components/Grouping/SubMenu'),
+      default: { render: h => h('router-view') }
+    },
+    beforeEnter (to, from, next) { redirectToFirstRoute(to, next) }
+  },
 ]
```

`src/Common/content/groupings/types.js` (so no caso com grouping):

```diff
+export const GROUPING_<MODULE>_NAME = '<Nome no menu>'
+export const GROUPING_<MODULE>_SHORT = '<Nome curto>'
+export const GROUPING_<MODULE>_SLUG = '/<slug>'
+export const GROUPING_<MODULE>_ICON = '<icone>'
```

`src/Common/content/groupings/index.js` (so no caso com grouping) — seguir o comentario inline do proprio arquivo, adicionar node em `baseGroupings` + valor em `groupingsList`:

```diff
 export const groupingsList = {
   ...
+  <Module>: '<slug-sem-barra>',
 }

 export const baseGroupings = [
   ...
+  {
+    name: types.GROUPING_<MODULE>_NAME,
+    short: types.GROUPING_<MODULE>_SHORT,
+    icon: types.GROUPING_<MODULE>_ICON,
+    path: types.GROUPING_<MODULE>_SLUG,
+    to: { path: types.GROUPING_<MODULE>_SLUG },
+    grouping: groupingsList.<Module>
+  },
 ]
```

## Referencias cruzadas

- [[Playbook/new-module]]
- [[Playbook/component-conventions]]
