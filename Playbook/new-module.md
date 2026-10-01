---
tags:
  - playbook
  - frontend
  - modulo
  - arquitetura
date: 2026-09-24
---

# Criar um Módulo Novo no SPA

## O que é

Essa documentação orienta o padrão de um boilerplate de como criar um novo módulo de primeiro nível do zero — estrutura de pastas e os pontos de wiring que conectam o módulo ao resto do app (aliases, store, rotas). Vale pros SPAs Convenia atuais (spa-admin, spa-colab) e pra qualquer SPA futura que siga a mesma base. Diferente de [[Playbook/playbook]] (que cobre um **card dentro** de um módulo existente que consome um organism), este guia é sobre abrir a área inteira.

O skeleton e os 4 pontos de wiring abaixo são o **padrão recomendado pra módulo novo** — não um inventário do que já existe. Módulos já criados em spa-admin e spa-colab variam entre si por débito orgânico (pasta a mais aqui, alias a menos ali); isso não é motivo pra desviar do padrão ao abrir um módulo novo.

Esse guia assume pouca familiaridade com o front. Se algum termo abaixo for novo, procure o glossário antes de seguir.

## Conceitos rápidos (glossário)

- **SPA** (Single Page Application) — o app carrega uma página só no navegador e troca o conteúdo em tela via JavaScript, sem recarregar o navegador a cada clique. `spa-admin` e `spa-colab` são dois SPAs diferentes da Convenia, cada um com seu próprio código, mas seguindo os mesmos padrões de arquitetura.
- **Módulo** — uma área grande e independente do app, tipo uma "seção" inteira (ex.: a área de folha de pagamento, a área de benefícios). Fica em `src/<NomeDoModulo>/`. Diferente de um **card**, que é uma peça pequena de tela dentro de um módulo já existente (esse caso está em [[Playbook/playbook]], não aqui).
- **Alias** — um atalho de import. Em vez de escrever um caminho relativo longo e frágil tipo `../../../store`, você escreve `@<Module>:store`. Quem resolve esse atalho é o `jsconfig.json` (ponto de wiring 1 abaixo).
- **Store (Vuex)** — o lugar centralizado onde o app guarda dados que várias telas precisam usar (ex.: dados do usuário logado, lista de itens carregada da API). Vuex é a biblioteca que a Convenia usa pra isso. Cada módulo tem seu pedaço de store, que precisa ser registrado na store raiz do app (ponto de wiring 2).
- **Rota** — o mapeamento entre uma URL (ex. `/pagina`) e o componente Vue que deve aparecer nela. Cada módulo declara suas próprias rotas, que depois são somadas às rotas do app inteiro (ponto de wiring 3).
- **Grouping** — quando um módulo tem uma seção própria no menu lateral do app (em vez de só uma rota solta sem menu visível). Envolve constantes extras além da rota (também no ponto de wiring 3).
- **Barrel** — um arquivo `index.js` que só reexporta outros arquivos da mesma pasta, servindo de "porta de entrada" única (`import algo from '@<Module>:services'` em vez de importar arquivo por arquivo). Útil em módulo pequeno; em módulo grande pode virar um arquivo gigante difícil de manter — nesse caso vale mais importar cada arquivo direto pelo path.
- **Eager vs lazy (loading)** — eager quer dizer que o código é carregado assim que o app sobe, sempre; lazy quer dizer que só é carregado quando de fato é usado (ex. ao entrar numa rota específica). Nesses SPAs, todo módulo registrado na store raiz é sempre eager — não existe um mecanismo de lazy-load de store por rota (ver ponto de wiring 2).
- **POM (Page Object Model)** — um jeito de organizar teste automatizado (Playwright, nesse caso) separando "o que a página tem" (seletores, ações) de "o que o teste verifica" (asserções), pra teste não quebrar toda vez que um detalhe visual muda.

## Skeleton de pastas

Isso é a estrutura de pastas que todo módulo novo deve ter dentro de `src/<Module>/`. Pense nela como o "esqueleto" que você copia e preenche — cada pasta tem uma responsabilidade específica, então evite misturar código de uma pasta em outra só por conveniência.

```
src/<Module>/
├── containers/          # smart components, Composition API — obrigatório
├── services/            # camada de API, contrato [err, data] — obrigatório
├── store/               # Vuex — obrigatório
│   ├── index.js         # barrel: export { default as <slice> } from '@<Module>:store/<Feature>'
│   ├── types.js         # TODAS as type constants do módulo, um arquivo só
│   └── <Feature>/index.js (ou <Feature>.js)   # um slice Vuex por feature
├── routes/              # obrigatório
│   ├── index.js         # export default — objeto de rota único OU array de sub-rotas
│   └── names.js         # (ou name.js) — constantes de nome de rota
├── components/          # dumb components — só se houver apresentação reusável dentro do módulo
├── content/
│   ├── consts/          # enums locais (Object.freeze) — ver Playbook/component-conventions
│   └── forms/           # schemas de form — quando o módulo tem forms próprios
├── helpers/             # funções puras do domínio do módulo
│   ├── index.js         # export default — objeto de rota único OU array de sub-rotas
├── composables/          # lógica reativa compartilhada entre containers do módulo
├── mixins/               # só se o módulo ainda tem código Options API legado
└── graphql/              # queries/mutations, só se o módulo consome GraphQL API legado
```

O que cada pasta guarda, em termos simples:
- **`containers/`** — os componentes "espertos", que buscam dado (via `services/` ou `store/`) e passam pra componentes de apresentação. É aqui que a tela de fato é montada.
- **`services/`** — toda chamada de API do módulo mora aqui. O contrato de retorno é sempre um array `[erro, dado]` (nunca lança exception pro container tratar; o container só olha se `erro` veio preenchido).
- **`store/`** — o estado do módulo em Vuex (o que foi carregado da API, o que o usuário está editando em tela, etc).
- **`routes/`** — quais URLs esse módulo responde e o que renderiza em cada uma.

`components/`, `content/`, `helpers/`, `composables/`, `mixins/`, `graphql/` são **opcionais** — adicione só quando o módulo realmente precisar. `containers/`, `services/`, `store/`, `routes/` são o mínimo que todo módulo tem.

Módulo pequeno (1-2 features) pode ter `services/index.js` como **barrel**, com um `import services from '@<Module>:services'` único nos containers. Módulo grande (4+ sub-features, cada uma com service próprio) geralmente **não** tem barrel — cada arquivo é importado pelo path direto (`@<Module>:services/<Feature>`), evitando um barrel gigante que reimporta tudo só pra usar uma função. Escolha conforme o tamanho, não force um barrel cedo demais.

## Os 4 pontos de wiring

"Wiring" aqui significa: além de criar as pastas acima, existem 4 lugares fora do módulo que precisam saber que ele existe. Esquecer um desses pontos é a causa mais comum de "criei o módulo mas não funciona" — o código existe, mas o app não sabe que ele existe.

### 1. `jsconfig.json` — aliases (único lugar a editar)

Todo import dentro do módulo usa alias (`@<Module>:...`) em vez de path relativo. Pra esse alias funcionar, ele precisa estar declarado no `jsconfig.json`. Sem esse passo, o import simplesmente não resolve e o build quebra.

Adicione as entradas do módulo em `jsconfig.json` → `compilerOptions.paths`, em ordem alfabética ascendente. Só inclua as entradas opcionais (`:components`, `:content`, `:helpers`, `:composables`) se o módulo de fato tiver essa pasta:

```jsonc
"@<Module>/*": ["src/<Module>/*"],
"@<Module>:components/*": ["src/<Module>/components/*"],
"@<Module>:composables/*": ["src/<Module>/composables/*"],
"@<Module>:containers/*": ["src/<Module>/containers/*"],
"@<Module>:content/*": ["src/<Module>/content/*"],
"@<Module>:helpers/*": ["src/<Module>/helpers/*"],
"@<Module>:routes/*": ["src/<Module>/routes/*"],
"@<Module>:services/*": ["src/<Module>/services/*"],
"@<Module>:store/*": ["src/<Module>/store/*"],
"@<Module>:types/*": ["src/<Module>/store/types/*"]
```

**Não mexa em `vite.config.js`.** Os aliases do Vite são gerados automaticamente a partir do `jsconfig.json` por `setViteAliases(__dirname)` (`@convenia/utils/modules/vite/setViteAliases`) — editar só o `jsconfig.json` já resolve build e IDE (autocomplete e "ir para definição" continuam funcionando).

### 2. `src/store.js` — registrar o módulo na store raiz

Cada módulo tem seu próprio pedaço de Vuex (`store/index.js`), mas isso sozinho não é suficiente — a store raiz do app inteiro (`src/store.js`) precisa importar e juntar esse pedaço, senão o Vuex do módulo simplesmente não existe em tempo de execução.

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

**Todo módulo registrado aqui fica sempre ativo** (eager) — não existe carregamento lazy por rota nesses SPAs (confirmado: nenhuma ocorrência de `meta.storeModules` no codebase). Não invente esse mecanismo achando que é um padrão — não é. Na prática isso significa: o estado do módulo é criado quando o app sobe, independente do usuário ter navegado pra lá ou não.

### 3. `routes/index.js` do módulo + `src/Common/routes/index.js`

O `routes/index.js` do módulo exporta:
- **um objeto de rota único**, se o módulo é uma rota standalone (ex. `<Module>` → `/<slug>`); ou
- **um array**, composto de arquivos irmãos por sub-feature (`routes/{featureA,featureB}.js`), se o módulo tem várias sub-rotas.

Em ambos os casos, um `routes/names.js` exporta as constantes de nome de rota (string), importado como `import * as names from '@<Module>:routes/names'` dentro do `routes/index.js`. Usar uma constante em vez de escrever a string direto evita erro de digitação e facilita renomear a rota depois.

Mas assim como a store, declarar a rota dentro do módulo não basta: `src/Common/routes/index.js` é o agregador central — importe o módulo lá e injete no array `routes[]`. Sem esse passo, a URL do módulo dá 404 mesmo o código estando certo. Dois padrões, escolha conforme o módulo:

**(a) Módulo standalone, sem menu próprio** — o módulo é acessado por URL direta, sem aparecer como uma seção no menu lateral:

```js
import <module> from '@<Module>:routes'
// ...
const routes = [
  ...
  <module>,
]
```

**(b) Módulo com seção própria no menu lateral** (grouping) — o módulo ganha um item visível no menu, com submenu próprio:

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

Os arquivos e o prefixo de constantes (`GROUPING_<MODULE>_*`) são os mesmos em spa-admin e spa-colab, mas o **shape** de cada entrada em `baseGroupings` diverge por SPA — cada uma tem sua própria convenção de slug (relativo vs path absoluto), de formato do `to:` (`{ name }` vs `{ path }`) e de campos de permissão/visibilidade. Não existe um shape universal: copie o formato das entradas irmãs já existentes no `groupings/index.js` do SPA onde o módulo está sendo criado, nunca o de outro SPA.

### 4. Store — shape interno e naming

`store/types.js` é um arquivo único pro módulo inteiro (não um por feature), com todas as constantes. Essas constantes são os "nomes" das getters/mutations/actions do Vuex — usar constante em vez de string solta evita erro de digitação silencioso (typo numa string não dá erro de build, só falha em runtime de um jeito confuso):

```js
export const <MODULE>_GET_<NOUN> = '<MODULE>/GET_<NOUN>'
export const <MODULE>_SET_<NOUN> = '<MODULE>/SET_<NOUN>'
```

`store/index.js` é o barrel que reúne um módulo Vuex por feature/slice (um "slice" é o pedaço de store referente a uma feature específica dentro do módulo):

```js
export { default as <sliceA> } from '@<Module>:store/<FeatureA>'
export { default as <sliceB> } from '@<Module>:store/<FeatureB>'
```

Cada slice (`store/<Feature>/index.js`) é um `{ state, getters, mutations, actions }` inline — não separe em `mutations.js`/`actions.js`/`state.js` por arquivo (exceção rara: extrair só o `state` pra um `state.js` sibling quando ele fica muito grande, ainda importado de volta no `index.js`).

## Testes

`tests/specs/<Module>/<Feature>/{data,pages,specs}` — mesma forma POM (data/pages/specs + `setup.js` + `<Group>Page.js`) já documentada em [[Templates/Codigo/spec.playwright]], só que a raiz é o módulo/feature, não um card isolado dentro de um módulo existente. Se o termo POM ainda não fizer sentido, ver o glossário no início desse guia e depois [[Templates/Codigo/spec.playwright]] pro exemplo completo.

## Red flags

Red flag = sinal de alerta de que algo foi feito fora do padrão documentado aqui. Se você se pegar fazendo qualquer um dos itens abaixo, pare e reveja:

- Inventar `meta.storeModules` ou qualquer carregamento lazy de store por rota — não existe nesses SPAs; todo módulo registrado em `src/store.js` é sempre eager.
- Editar `vite.config.js` pra adicionar alias — os aliases vêm do `jsconfig.json` via `setViteAliases`, nunca duplique em `vite.config.js`.
- Criar as constantes `GROUPING_<MODULE>_*` em outro lugar que não `src/Common/content/groupings/types.js`, ou registrar o grouping fora de `baseGroupings`/`groupingsList` em `src/Common/content/groupings/index.js`.
- Separar `store/types.js` por feature dentro do módulo — é um arquivo único pro módulo inteiro.
- Dividir um slice de store em `state.js`/`mutations.js`/`actions.js` por padrão — a forma default é um `index.js` só, com exceção pontual pro `state.js` quando ele cresce muito.
- Adicionar `services/index.js` barrel num módulo grande só por convenção — avalie se o módulo realmente se beneficia, ou se import direto por path serve melhor (é o padrão dos módulos maiores hoje).

## Referências cruzadas

- [[Playbook/playbook]]
- [[Playbook/component-conventions]]
- [[Templates/Codigo/module-scaffold]]
- [[Templates/Codigo/spec.playwright]]
