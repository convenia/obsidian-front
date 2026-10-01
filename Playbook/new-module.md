---
tags:
  - playbook
  - frontend
  - modulo
  - arquitetura
date: 2026-10-01
---

# Criar um Módulo Novo no SPA

## O que é

Como criar uma **área de primeiro nível** do SPA (ex.: `src/Board/`, `src/Information/`, `src/Team/`) do zero — estrutura de pastas e os pontos de wiring que conectam o módulo ao resto do app (aliases, store, rotas). Diferente de [[Playbook/playbook]] (que cobre um **card dentro** de um módulo existente que consome um organism), este guia é sobre abrir a área inteira.

> Não existe doc oficial "como adicionar um módulo" no repo — a convenção é implícita, inferida comparando módulos existentes. Este guia foi escrito lendo o repo real (`src/Board/`, `src/Information/`, `src/Team/`, `src/store.js`, `src/Common/routes/index.js`, `jsconfig.json`).

## Skeleton de pastas

```
src/<Module>/
├── containers/          # smart components, Composition API — obrigatório
├── services/            # camada de API, contrato [err, data] — obrigatório
├── store/               # Vuex — obrigatório
│   ├── index.js         # barrel — só spa-colab: export { default as <slice> } from '@<Module>:store/<Feature>'
│   ├── types.js         # TODAS as type constants do módulo, um arquivo só
│   └── <Feature>/index.js (ou <Feature>.js)   # um slice Vuex por feature
├── routes/              # obrigatório
│   ├── index.js         # export default — objeto de rota único OU array de sub-rotas
│   └── names.js         # (ou name.js) — constantes de nome de rota
├── components/          # dumb components — só se houver apresentação reusável dentro do módulo
├── content/
│   ├── consts/          # enums locais (Object.freeze) — ver Playbook/component-conventions
│   ├── forms/           # schemas de form — quando o módulo tem forms próprios
│   └── mappers/         # map<Entity> + map<Entity>Input — ver Playbook/mappers
├── helpers/             # funções puras do domínio do módulo
├── composables/          # lógica reativa compartilhada entre containers do módulo
├── mixins/               # só se o módulo ainda tem código Options API legado
└── graphql/              # queries/mutations, só se o módulo consome GraphQL
```

`components/`, `content/`, `helpers/`, `composables/`, `mixins/`, `graphql/` são **opcionais** — adicione só quando o módulo realmente precisar. `containers/`, `services/`, `store/`, `routes/` são o mínimo que todo módulo tem.

Módulo pequeno (poucas features, ex. `Board`) pode ter `services/index.js` como **barrel**. Módulo grande com muitas sub-features (ex. `Information`, `Team`) geralmente **não** tem barrel — cada arquivo de service é importado pelo path direto (`@Module:services/<Feature>/<Card>`). Escolha conforme o tamanho, não force um barrel cedo demais.

## Os 4 pontos de wiring

### 1. `jsconfig.json` — aliases (único lugar a editar)

Adicione as entradas do módulo em `jsconfig.json` → `compilerOptions.paths`:

```jsonc
"@<Module>/*": ["src/<Module>/*"],
"@<Module>:containers/*": ["src/<Module>/containers/*"],
"@<Module>:services/*": ["src/<Module>/services/*"],
"@<Module>:store/*": ["src/<Module>/store/*"],
"@<Module>:types/*": ["src/<Module>/store/types/*"],
"@<Module>:routes/*": ["src/<Module>/routes/*"]
// + :components, :content, :helpers, :composables conforme o módulo usar
```

**Não mexa em `vite.config.js`.** Os aliases do Vite são gerados automaticamente a partir do `jsconfig.json` por `setViteAliases(__dirname)` (`@convenia/utils/modules/vite/setViteAliases`) — editar só o `jsconfig.json` já resolve build e IDE.

### 2. Registrar os slices de store

O mecanismo difere por SPA. Ver tabela completa em [[Playbook/store]].

**spa-colab — eager via `src/store.js`:**

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

No spa-colab, **todo módulo registrado aqui fica sempre ativo** (eager) — não existe `meta.storeModules` nesse repo.

**spa-admin — lazy por rota via `meta.storeModules`:**

`src/store.js` do spa-admin só registra `...common`. Slice de feature MUST ser declarado na rota que o usa; o guard de rota chama `registerStoreModules` (`src/Common/modules/router/helpers.js`), que faz `store.registerModule` se o módulo ainda não existe. No spa-admin o módulo **não** tem `store/index.js`.

```js
// src/<Module>/routes/index.js
meta: {
  storeModules: {
    <slice>: () => import('@<Module>:store/<Feature>'),
  },
},
```

### 3. `routes/index.js` do módulo + `src/Common/routes/index.js`

O `routes/index.js` do módulo exporta:
- **um objeto de rota único**, se o módulo é uma rota standalone (ex. `Board` → `/painel`); ou
- **um array**, composto de arquivos irmãos por sub-feature (`routes/{featureA,featureB}.js`), se o módulo tem várias sub-rotas.

Em ambos os casos, um `routes/names.js` exporta as constantes de nome de rota (string), importado como `import * as names from '@<Module>:routes/names'` dentro do `routes/index.js`.

`src/Common/routes/index.js` é o agregador central — importe o módulo lá e injete no array `routes[]`. Dois padrões, escolha conforme o módulo:

**(a) Módulo standalone, sem menu próprio** (ex. `Board`):

```js
import <module> from '@<Module>:routes'
// ...
const routes = [
  ...
  <module>,
]
```

**(b) Módulo com seção própria no menu lateral** (grouping):

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

Pro caso (b), o módulo precisa de duas constantes a mais, em arquivos que **já existem** (não crie um lugar novo pra elas):
- `src/Common/content/groupings/types.js` — `GROUPING_<MODULE>_NAME`, `_SHORT`, `_SLUG`, `_ICON`.
- `src/Common/content/groupings/index.js` — um node novo em `baseGroupings` + o valor correspondente em `groupingsList`. Esse arquivo **já tem um comentário inline** explicando o passo a passo — siga ele.

### 4. Store — shape interno e naming

`store/types.js` é um arquivo único pro módulo inteiro (não um por feature), com todas as constantes. Naming e valor seguem [[Playbook/store-types]]:

```js
export * from '@types'

export const <TYPE_PREFIX>_<FEATURE>_GET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_GET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_SET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_SET_<AREAS>'
```

No spa-colab, `store/index.js` é o barrel que reúne um módulo Vuex por feature/slice:

```js
export { default as <sliceA> } from '@<Module>:store/<FeatureA>'
export { default as <sliceB> } from '@<Module>:store/<FeatureB>'
```

Cada slice (`store/<Feature>/index.js`) é um `{ state, getters, mutations, actions }` inline — não separe em `mutations.js`/`actions.js`/`state.js` por arquivo (exceção rara: extrair só o `state` pra um `state.js` sibling quando ele fica muito grande, ainda importado de volta no `index.js`).

## Testes

`tests/specs/<Module>/<Feature>/{data,pages,specs}` — mesma forma POM (data/pages/specs + `setup.js` + `<Group>Page.js`) já documentada em [[Templates/Codigo/spec.playwright]], só que a raiz é o módulo/feature, não um card isolado dentro de um módulo existente.

## Red flags

- Usar o mecanismo de registro da outra SPA: `meta.storeModules` no spa-colab (lá todo módulo é eager via `src/store.js`) ou barrel + `src/store.js` no spa-admin (lá é lazy por rota).
- Editar `vite.config.js` pra adicionar alias — os aliases vêm do `jsconfig.json` via `setViteAliases`, nunca duplique em `vite.config.js`.
- Criar as constantes `GROUPING_<MODULE>_*` em outro lugar que não `src/Common/content/groupings/types.js`, ou registrar o grouping fora de `baseGroupings`/`groupingsList` em `src/Common/content/groupings/index.js`.
- Separar `store/types.js` por feature dentro do módulo — é um arquivo único pro módulo inteiro.
- Dividir um slice de store em `state.js`/`mutations.js`/`actions.js` por padrão — a forma default é um `index.js` só, com exceção pontual pro `state.js` quando ele cresce muito.
- Adicionar `services/index.js` barrel num módulo grande só por convenção — avalie se o módulo realmente se beneficia, ou se import direto por path serve melhor (é o padrão dos módulos maiores hoje).

## Referências cruzadas

- [[Playbook/playbook]]
- [[Playbook/component-conventions]]
- [[Playbook/store-types]]
- [[Playbook/store]]
- [[Templates/Codigo/module-scaffold]]
- [[Templates/Codigo/spec.playwright]]
