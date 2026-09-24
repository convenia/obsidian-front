---
tags:
  - playbook
  - frontend
  - checklist
date: 2026-09-24
---

# Red Flags — Parar

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O "core compartilhado" vale pra qualquer visao; a secao "especifico por visao" vale so onde o token bate.

## O que e

A lista de restricoes de uma migracao de card. Cada item e uma violacao — se voce esta prestes a fazer algum destes, pare e corrija.

## Shape do container

- Container novo em Options API em vez de `<script setup>`.
- Montar schema de form ou `CFormBuilderModal` no container.
- Guardar estado de modal aberto/fechado, edit/create ou arquivo removido no container.
- Container escolhendo create vs update em vez de despachar a unica action de update e deixar a store decidir.
- Renderizar varios cards via `cards` computed em vez do organism iterar.
- **Drift de naming** — arquivo/`name` do container, arquivo/funcoes de service, ou types de store usando um token de dominio diferente do fragment/`map<Domain>Input` do organism.
- **Drift de pasta `<Area>`** — `<Area>` resolvido pra um nome em `containers/<Area>/` e outro em `services/<Area>/`, `store/<Area>/` ou no spec root de teste. `<Area>` e um unico token; renomeie toda superficie junto, nunca uma isolada.

## Fonte de fields

- Manter o endpoint legado de fields pra qualquer sub-area apos migrar — sem hibrido.
- Buscar system fields e custom fields como duas rotas ou duas actions separadas em vez do fetch unificado.
- Adicionar fetch de options ou entrada `fieldsOptions` pra um select cujas options ja vem no payload de `systemFields`.

## Mappers e payload

- Definir mapper de leitura ou `pickSection` **na SPA** em vez do `content/mappers` do organism.
- Montar payload na SPA — criar `services/<Area>/mappers.js` ou chamar `buildPayloadWithCustomFields` da SPA.
- Destructurar `.schema` de `customFields` (passe o objeto id-keyed inteiro).
- Filtrar `invalidFields` com `delete` no spread em vez de omitir por condicao no mapper.

## Shape de store

- Uma action/type `CREATE_*` separada pra um card que ja tem a action `UPDATE` unica.
- Manter actions de upload/delete de arquivo separadas em vez da unica action de files.
- Type constant fora da convencao `<TYPE_PREFIX>_<VERB>_[<AREA>_]<ENTITY>`.
- Deixar `state`/`mutations`/`actions`/types fora de ordem alfabetica estrita — inclusive entradas pre-existentes no arquivo tocado.

## Options e dados dependentes

- Cidades via `common/CITIES`/`cityState` em vez da action de options do modulo.
- Chamar a query GraphQL legada de cidade em vez do endpoint REST.
- Colocar `provide` de `getCities`/`getZipCode`/`getFileUrl` no loader **pai** em vez do container filho.

## Caminhos legados removidos

- Reativar `resolveCustomFieldValues` numa sub-area ja migrada.
- Manter a chamada de `requiredSystemFields` numa area migrada.

## Teste

- Teste 2 cobrindo so um subconjunto de tipos de custom field em vez de um por tipo.
- Inventar shape/valor de custom field na fixture em vez de reusar os mocks de Storybook do organism.
- Hoistar um testid do Playwright pra `const`/objeto e interpolar em vez de inline literal.

## Especifico por visao

- **`<HAS_ADMISSION>` = sim (spa-admin):** renomear um type de custom field compartilhado sem dar grep nos dispatchers cross-feature.
- **`<VISAO>` = route param (familia colab):** fixar um segmento de role em vez de usar o param `route`.
- **`<CAN_EDIT_EXPR>` (familia colab):** usar `!isDisconnected` (isso e admin) ou config de store em vez do `true` fixo no card V2.
- **familia colab:** adicionar fetch novo no path legado `Common`/`Information` em vez de migrar pro service V2.
- **`<BRANCH_GATE>` explicito (spa-colab-self):** conectar o container antes de confirmar que o branch do submodule realmente contem o organism.

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/container]]
- [[Playbook/store]]
- [[Playbook/testing]]
- [[Playbook/cleanup]]
