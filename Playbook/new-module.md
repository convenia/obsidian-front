---
tags:
  - playbook
  - frontend
  - modulo
  - arquitetura
date: 2026-09-24
---

# Criar um Modulo Novo no SPA

## O que e

Como criar uma **area de primeiro nivel** do SPA (ex.: `src/Board/`, `src/Information/`, `src/Team/`) do zero — estrutura de pastas e os pontos de wiring que conectam o modulo ao resto do app (aliases, store, rotas). Diferente de [[Playbook/playbook]] (que cobre um **card dentro** de um modulo existente que consome um organism), este guia e sobre abrir a area inteira.

> Nao existe doc oficial "como adicionar um modulo" no repo — a convencao e implicita, inferida comparando modulos existentes. Este guia foi escrito lendo o repo real (`src/Board/`, `src/Information/`, `src/Team/`, `src/store.js`, `src/Common/routes/index.js`, `jsconfig.json`).

## Skeleton de pastas

```
src/<Module>/
├── containers/          # smart components, Composition API — obrigatorio
├── services/            # camada de API, contrato [err, data] — obrigatorio
├── store/               # Vuex — obrigatorio
│   ├── index.js         # barrel: export { default as <slice> } from '@<Module>:store/<Feature>'
│   ├── types.js         # TODAS as type constants do modulo, um arquivo so
│   └── <Feature>/index.js (ou <Feature>.js)   # um slice Vuex por feature
├── routes/              # obrigatorio
│   ├── index.js         # export default — objeto de rota unico OU array de sub-rotas
│   └── names.js         # (ou name.js) — constantes de nome de rota
├── components/          # dumb components — so se houver apresentacao reusavel dentro do modulo
├── content/
│   ├── consts/          # enums locais (Object.freeze) — ver Playbook/component-conventions
│   └── forms/           # schemas de form — quando o modulo tem forms proprios
├── helpers/             # funcoes puras do dominio do modulo
├── composables/          # logica reativa compartilhada entre containers do modulo
├── mixins/               # so se o modulo ainda tem codigo Options API legado
└── graphql/              # queries/mutations, so se o modulo consome GraphQL
```

`components/`, `content/`, `helpers/`, `composables/`, `mixins/`, `graphql/` sao **opcionais** — adicione so quando o modulo realmente precisar. `containers/`, `services/`, `store/`, `routes/` sao o minimo que todo modulo tem.

Modulo pequeno (poucas features, ex. `Board`) pode ter `services/index.js` como **barrel**. Modulo grande com muitas sub-features (ex. `Information`, `Team`) geralmente **nao** tem barrel — cada arquivo de service e importado pelo path direto (`@Module:services/<Feature>/<Card>`). Escolha conforme o tamanho, nao force um barrel cedo demais.

## Os 4 pontos de wiring

### 1. `jsconfig.json` — aliases (unico lugar a editar)

Adicione as entradas do modulo em `jsconfig.json` → `compilerOptions.paths`:

```jsonc
"@<Module>/*": ["src/<Module>/*"],
"@<Module>:containers/*": ["src/<Module>/containers/*"],
"@<Module>:services/*": ["src/<Module>/services/*"],
"@<Module>:store/*": ["src/<Module>/store/*"],
"@<Module>:types/*": ["src/<Module>/store/types/*"],
"@<Module>:routes/*": ["src/<Module>/routes/*"]
// + :components, :content, :helpers, :composables conforme o modulo usar
```

**Nao mexa em `vite.config.js`.** Os aliases do Vite sao gerados automaticamente a partir do `jsconfig.json` por `setViteAliases(__dirname)` (`@convenia/utils/modules/vite/setViteAliases`) — editar so o `jsconfig.json` ja resolve build e IDE.

### 2. `src/store.js` — registrar o modulo na store raiz

```js
import * as <module> from '@<Module>:store'
// ...
export default new Vuex.Store({
  modules: {
    // ...
    ...<module>,
  }
})
```

**Todo modulo registrado aqui fica sempre ativo** (eager) — nao existe carregamento lazy por rota neste repo (confirmado: nenhuma ocorrencia de `meta.storeModules` no codebase). Nao invente esse mecanismo achando que e um padrao — nao e.

### 3. `routes/index.js` do modulo + `src/Common/routes/index.js`

O `routes/index.js` do modulo exporta:
- **um objeto de rota unico**, se o modulo e uma rota standalone (ex. `Board` → `/painel`); ou
- **um array**, composto de arquivos irmãos por sub-feature (`routes/{featureA,featureB}.js`), se o modulo tem varias sub-rotas.

Em ambos os casos, um `routes/names.js` exporta as constantes de nome de rota (string), importado como `import * as names from '@<Module>:routes/names'` dentro do `routes/index.js`.

`src/Common/routes/index.js` e o agregador central — importe o modulo la e injete no array `routes[]`. Dois padroes, escolha conforme o modulo:

**(a) Modulo standalone, sem menu proprio** (ex. `Board`):

```js
import <module> from '@<Module>:routes'
// ...
const routes = [
  ...
  <module>,
]
```

**(b) Modulo com secao propria no menu lateral** (grouping):

```js
import <module> from '@<Module>:routes'
// ...
{
  path: types.GROUPING_<MODULE>_SLUG,
  meta: { grouping: groupingsList.<Module> },
  children: <module>,
  components: {
    subMenu: () => import('@components/Grouping/SubMenu'),
    default: { render: h => h('router-view') }
  },
  beforeEnter (to, from, next) { redirectToFirstRoute(to, next) }
}
```

Pro caso (b), o modulo precisa de duas constantes a mais, em arquivos que **ja existem** (nao crie um lugar novo pra elas):
- `src/Common/content/groupings/types.js` — `GROUPING_<MODULE>_NAME`, `_SHORT`, `_SLUG`, `_ICON`.
- `src/Common/content/groupings/index.js` — um node novo em `baseGroupings` + o valor correspondente em `groupingsList`. Esse arquivo **ja tem um comentario inline** explicando o passo a passo — siga ele.

### 4. Store — shape interno e naming

`store/types.js` e um arquivo unico pro modulo inteiro (nao um por feature), com todas as constantes:

```js
export const <MODULE>_GET_<NOUN> = '<MODULE>/GET_<NOUN>'
export const <MODULE>_SET_<NOUN> = '<MODULE>/SET_<NOUN>'
```

`store/index.js` e o barrel que reune um modulo Vuex por feature/slice:

```js
export { default as <sliceA> } from '@<Module>:store/<FeatureA>'
export { default as <sliceB> } from '@<Module>:store/<FeatureB>'
```

Cada slice (`store/<Feature>/index.js`) e um `{ state, getters, mutations, actions }` inline — nao separe em `mutations.js`/`actions.js`/`state.js` por arquivo (excecao rara: extrair so o `state` pra um `state.js` sibling quando ele fica muito grande, ainda importado de volta no `index.js`).

## Testes

`tests/specs/<Module>/<Feature>/{data,pages,specs}` — mesma forma POM (data/pages/specs + `setup.js` + `<Group>Page.js`) ja documentada em [[Templates/Codigo/spec.playwright]], so que a raiz e o modulo/feature, nao um card isolado dentro de um modulo existente.

## Red flags

- Inventar `meta.storeModules` ou qualquer carregamento lazy de store por rota — nao existe nesse repo; todo modulo registrado em `src/store.js` e sempre eager.
- Editar `vite.config.js` pra adicionar alias — os aliases vem do `jsconfig.json` via `setViteAliases`, nunca duplique em `vite.config.js`.
- Criar as constantes `GROUPING_<MODULE>_*` em outro lugar que nao `src/Common/content/groupings/types.js`, ou registrar o grouping fora de `baseGroupings`/`groupingsList` em `src/Common/content/groupings/index.js`.
- Separar `store/types.js` por feature dentro do modulo — e um arquivo unico pro modulo inteiro.
- Dividir um slice de store em `state.js`/`mutations.js`/`actions.js` por padrao — a forma default e um `index.js` so, com excecao pontual pro `state.js` quando ele cresce muito.
- Adicionar `services/index.js` barrel num modulo grande so por convencao — avalie se o modulo realmente se beneficia, ou se import direto por path serve melhor (e o padrao dos modulos maiores hoje).

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/component-conventions]]
- [[Templates/Codigo/module-scaffold]]
- [[Templates/Codigo/spec.playwright]]
