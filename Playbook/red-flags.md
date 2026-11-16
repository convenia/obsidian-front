---
tags:
  - playbook
  - frontend
  - checklist
date: 2026-11-16
---

# Red Flags — Parar

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O "core compartilhado" vale pra qualquer visao e qualquer card; a secao "especifico por visao" vale so onde o token bate.

## O que e

A lista de restricoes de uma migracao de card. Cada item e uma violacao — se voce esta prestes a fazer algum destes, pare e corrija. Serve tambem de checklist de review: ver [[Playbook/review]].

## Como ler as tabelas

- **Sev** — peso do item num review:
  - `bloqueia` — quebra o padrao de um jeito que gera bug, retrabalho ou divergencia entre visoes.
  - `corrigir` — desvio real, mas que sozinho nao justifica outra rodada de review.
  - `sugestao` — consistencia; nao segura PR.
- **Como detectar** — por onde comecar a procurar no diff. E um ponto de partida, nao prova: abra o arquivo antes de apontar.
- **Fonte** — trecho do playbook da Convenia que sustenta o item (caminhos relativos a `convencoes-de-desenvolvimento/front-end/`, ver [[Playbook/fontes]]), ou `so vault` quando a regra so existe aqui. Item `so vault` vale com a severidade indicada dentro do padrao card+organism; fora dele e sugestao.

## Shape do container

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Container novo em Options API em vez de `<script setup>`. | bloqueia | Arquivo novo em `containers/` com `export default {` e sem `<script setup>`. | `introducao/composition-api.md:7-12` |
| Montar schema de form ou form primitive de modal no container. | bloqueia | `schema`, `c-form-builder` ou `c-form-builder-modal` dentro de `containers/`. | so vault |
| Guardar estado de modal aberto/fechado, edit/create ou arquivo removido no container. | bloqueia | `ref(false)` com nome `isOpen`/`isEditing`/`isCreating`/`removedFile` em `containers/`. | so vault |
| Container escolhendo create vs update em vez de despachar a unica action de update e deixar a store decidir. | bloqueia | `if (id)` ou ternario escolhendo entre duas actions no handler de `@submit`. | so vault |
| Renderizar varios cards via `cards` computed em vez do organism iterar. | corrigir | `v-for` sobre um computed de cards no template do container. | so vault |
| **Drift de naming** — arquivo/`name` do container, arquivo/funcoes de service, ou types de store usando um token de dominio diferente do fragment/mapper de escrita do organism. | corrigir | Comparar o nome do fragment do organism com o arquivo do container, as funcoes do service e os types. | so vault |
| **Drift de pasta `<Area>`** — `<Area>` resolvido pra um nome em `containers/<Area>/` e outro em `services/<Area>/`, `store/<Area>/` ou no spec root de teste. `<Area>` e um unico token; renomeie toda superficie junto, nunca uma isolada. | corrigir | Listar as quatro pastas tocadas pelo PR e comparar o segmento `<Area>`. | so vault |
| Logica de negocio ou acesso a store dentro de componente de apresentacao. | bloqueia | `mapState`/`mapActions`/`useStore` ou chamada de service em `components/`. | `introducao/container-component.md:7-25` |

## Fonte de metadado

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Manter o fetch legado de metadado (uma rota por pedaco do metadado, ou um mapper de leitura que monta o metadado inline na SPA) pra qualquer sub-area apos migrar — sem hibrido. | bloqueia | Action/service legado de metadado ainda chamado por um container ja migrado. | so vault |
| Buscar o metadado do card como duas rotas ou duas actions separadas em vez do fetch unificado. | bloqueia | Dois dispatches de metadado no mesmo loader. | so vault |
| Adicionar fetch de options ou entrada de options separada pra um select cujas options ja vem no metadado. | corrigir | Action `get*Options` nova pra um campo que ja traz `options` no metadado. | so vault |

## Mappers e payload

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Definir mapper de leitura ou slicing de secao **na SPA** em vez do organism. | bloqueia | Arquivo `mappers.js` novo em `services/` ou `store/` da SPA. | so vault |
| Montar payload na SPA — criar `services/<Area>/mappers.js` ou montar o payload a mao em vez de importar o mapper de escrita do organism. | bloqueia | Objeto de payload montado campo a campo no service ou no container. | so vault |
| Destructurar so uma parte do metadado quando o mapper de escrita espera o objeto inteiro daquela sub-area. | bloqueia | Destructuring do metadado logo antes da chamada do mapper de escrita. | so vault |
| Filtrar `invalidFields` com `delete` no spread em vez de omitir por condicao no mapper. | corrigir | `delete payload.` perto de `invalidFields`. | so vault |

## Shape de store

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Uma action/type `CREATE_*` separada pra um card que ja tem a action `UPDATE` unica. | bloqueia | Type `*_CREATE_*` novo ao lado de um `*_UPDATE_*` da mesma entidade. | so vault |
| Manter actions de upload/delete de arquivo separadas em vez da unica action de files. | corrigir | Actions `upload*`/`delete*File` separadas pra mesma entidade. | so vault |
| Type constant fora da convencao `<TYPE_PREFIX>_<VERB>_[<AREA>_]<ENTITY>`. | corrigir | Ler os types adicionados no diff de `types.js`. | so vault |
| Deixar `state`/`mutations`/`actions`/types fora de ordem alfabetica estrita — inclusive entradas pre-existentes no arquivo tocado. | sugestao | Ler as chaves do arquivo tocado na ordem em que aparecem. | so vault |
| Action que nao devolve `[ err, data ]` ou engole o erro do service. | bloqueia | Action sem `return [ err, data ]`, ou `err` lido e descartado. | `introducao/referencia-arquitetura.md:631-727` |

## Options e dados dependentes

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Buscar uma option dependente por um modulo/rota generico em vez da action de options dedicada daquela area. | corrigir | Dispatch pra modulo de outra area dentro do container do card. | so vault |
| Chamar uma query GraphQL legada de option em vez do endpoint REST oficial. | bloqueia | Query GraphQL de option adicionada ou mantida no service da area. | so vault |
| Colocar `provide` de uma option dependente ou `getFileUrl` no loader **pai** em vez do container filho. | corrigir | `provide(` no loader pai com chave de option do card. | so vault |

## Caminhos legados removidos

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Reativar resolucao de valor de campo dinamico numa sub-area ja migrada. | bloqueia | Helper legado de resolucao de campo importado de novo num arquivo migrado. | so vault |
| Manter a chamada de obrigatoriedade separada numa area migrada (mandatoriness ja vem no payload de metadado). | bloqueia | Action/service de obrigatoriedade ainda despachado no loader. | so vault |

## Falha de request

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Request de listagem que falha e a tela mostra empty state, como se nao houvesse itens. | bloqueia | Empty state condicionado so ao tamanho da lista, sem olhar `err`. | `processos-produto/falhas-graves.md:15-17` |
| Erro de service lido e nao mostrado ao usuario. | bloqueia | `const [ err ] = await` sem feedback nem `callback(err)` no caminho de erro. | `testes/escolha.md:13-14` |

## Teste

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Teste 2 cobrindo so um subconjunto dos tipos de campo em vez de um por tipo suportado pelo organism. | corrigir | Comparar os tipos na fixture de metadado com os tipos que o organism suporta. | so vault |
| Inventar shape/valor de campo na fixture em vez de reusar os mocks de Storybook do organism. | corrigir | Fixture nova sem correspondente nos mocks de story do organism. | so vault |
| Hoistar um testid do Playwright pra `const`/objeto e interpolar em vez de inline literal. | sugestao | `getByTestId(` com template string ou variavel. | so vault |
| Grupo de integracao sem nenhum teste `@always`. | bloqueia | `grep -L "@always"` nos specs do grupo. | `testes/integracao.md:398-400` |
| Fluxo que fala com API sem teste do cenario de falha. | corrigir | Spec sem nenhuma fixture `*Error` nem mock com status de erro. | `testes/integracao.md:296-391` |
| Teste unitario de service, ou de wiring que o Playwright ja cobre. | corrigir | Arquivo `*.test.js` novo importando de `services/`. | `testes/escolha.md:15-17` |

## Componente/organism (ver [[Playbook/component-conventions]])

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Props/emits fora de ordem alfabetica ascendente. | sugestao | Ler as chaves de `defineProps`/`defineEmits` na ordem. | so vault |
| Atributos de template fora da ordem (estrutural → alfabetico → eventos). | sugestao | Ler os atributos das tags tocadas no diff. | so vault |
| Comentario no codigo alem das duas excecoes permitidas (JSDoc/constraint nao-obvia). | sugestao | Linhas `//` ou `<!--` adicionadas no diff. O playbook pede "comente o porque, nao o que" — a regra mais restrita daqui esta em aberto em [[Playbook/fontes]]. | `introducao/padroes-javascript.md:9` |
| Organism importado por path profundo em vez do barrel raiz do pacote `@convenia/<area>-organisms`. | corrigir | `from '@convenia/` seguido de `-organisms/` com subpasta. | so vault |
| `@convenia/components` importado pelo barrel em vez do caminho profundo. | corrigir | `import {` seguido de `} from '@convenia/components'`. | `lib-compartilhada/requisitos-tecnicos.md:14-21` |
| Const de enum sem `Object.freeze`. | sugestao | `export const` em maiusculas atribuindo objeto literal sem `Object.freeze`. | so vault |
| Prop de entidade nomeada genericamente (`values`) em vez do substantivo de dominio. | corrigir | `values`/`data`/`item` em `defineProps` de um card. | so vault |
| `<style scoped>` em codigo de feature da SPA, ou componente novo da lib sem `scoped`. | corrigir | `<style` nos arquivos `.vue` do diff, conferindo se o arquivo e de SPA ou de `.@convenia`. | `introducao/css.md:27-31`, `lib-compartilhada/requisitos-tecnicos.md:23-36` |

## Especifico por visao

| Red flag | Sev | Como detectar | Fonte |
|---|---|---|---|
| Renomear um type de metadado compartilhado sem dar grep nos dispatchers cross-feature, quando essa visao tem uma trilha de integracao adjacente que depende dele. | bloqueia | Grep do type antigo no repo inteiro, nao so na pasta da area. | so vault |
| Fixar um segmento de role/visao em vez de usar o param que a visao expoe pra isso. | bloqueia | String literal de role/visao em URL de service ou rota. | so vault |
| Usar a expressao de `canEdit` de uma visao diferente da que esta sendo trabalhada. | bloqueia | Comparar `canEdit` do container com `<CAN_EDIT_EXPR>` do profile da visao. | so vault |
| Adicionar fetch novo num path legado em vez de migrar pro service V2 daquela visao. | corrigir | Funcao nova num arquivo de service legado. | so vault |
| Conectar o container antes de confirmar que o branch do submodule realmente contem o organism, quando essa visao tiver esse gate explicito. | bloqueia | `git -C .@convenia log` no commit apontado pelo PR, procurando o organism. | so vault |

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/review]]
- [[Playbook/fontes]]
- [[Playbook/container]]
- [[Playbook/store]]
- [[Playbook/testing]]
- [[Playbook/cleanup]]
