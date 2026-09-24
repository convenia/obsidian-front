---
tags:
  - playbook
  - frontend
  - checklist
date: 2026-09-24
---

# Red Flags — Parar

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O "core compartilhado" vale pra qualquer visao e qualquer card; a secao "especifico por visao" vale so onde o token bate.

## O que e

A lista de restricoes de uma migracao de card. Cada item e uma violacao — se voce esta prestes a fazer algum destes, pare e corrija.

## Shape do container

- Container novo em Options API em vez de `<script setup>`.
- Montar schema de form ou form primitive de modal no container.
- Guardar estado de modal aberto/fechado, edit/create ou arquivo removido no container.
- Container escolhendo create vs update em vez de despachar a unica action de update e deixar a store decidir.
- Renderizar varios cards via `cards` computed em vez do organism iterar.
- **Drift de naming** — arquivo/`name` do container, arquivo/funcoes de service, ou types de store usando um token de dominio diferente do fragment/mapper de escrita do organism.
- **Drift de pasta `<Area>`** — `<Area>` resolvido pra um nome em `containers/<Area>/` e outro em `services/<Area>/`, `store/<Area>/` ou no spec root de teste. `<Area>` e um unico token; renomeie toda superficie junto, nunca uma isolada.

## Fonte de metadado

- Manter o fetch legado de metadado (uma rota por pedaco do metadado, ou um mapper de leitura que monta o metadado inline na SPA) pra qualquer sub-area apos migrar — sem hibrido.
- Buscar o metadado do card como duas rotas ou duas actions separadas em vez do fetch unificado.
- Adicionar fetch de options ou entrada de options separada pra um select cujas options ja vem no metadado.

## Mappers e payload

- Definir mapper de leitura ou slicing de secao **na SPA** em vez do organism.
- Montar payload na SPA — criar `services/<Area>/mappers.js` ou montar o payload a mao em vez de importar o mapper de escrita do organism.
- Destructurar so uma parte do metadado quando o mapper de escrita espera o objeto inteiro daquela sub-area.
- Filtrar `invalidFields` com `delete` no spread em vez de omitir por condicao no mapper.

## Shape de store

- Uma action/type `CREATE_*` separada pra um card que ja tem a action `UPDATE` unica.
- Manter actions de upload/delete de arquivo separadas em vez da unica action de files.
- Type constant fora da convencao `<TYPE_PREFIX>_<VERB>_[<AREA>_]<ENTITY>`.
- Deixar `state`/`mutations`/`actions`/types fora de ordem alfabetica estrita — inclusive entradas pre-existentes no arquivo tocado.

## Options e dados dependentes

- Buscar uma option dependente por um modulo/rota generico em vez da action de options dedicada daquela area.
- Chamar uma query GraphQL legada de option em vez do endpoint REST oficial.
- Colocar `provide` de uma option dependente ou `getFileUrl` no loader **pai** em vez do container filho.

## Caminhos legados removidos

- Reativar resolucao de valor de campo dinamico numa sub-area ja migrada.
- Manter a chamada de obrigatoriedade separada numa area migrada (mandatoriness ja vem no payload de metadado).

## Teste

- Teste 2 cobrindo so um subconjunto dos tipos de campo em vez de um por tipo suportado pelo organism.
- Inventar shape/valor de campo na fixture em vez de reusar os mocks de Storybook do organism.
- Hoistar um testid do Playwright pra `const`/objeto e interpolar em vez de inline literal.

## Componente/organism (ver [[Playbook/component-conventions]])

- Props/emits fora de ordem alfabetica ascendente.
- Atributos de template fora da ordem (estrutural → alfabetico → eventos).
- Comentario no codigo alem das duas excecoes permitidas (JSDoc/constraint nao-obvia).
- Import de path profundo em vez do barrel raiz do pacote.
- Const de enum sem `Object.freeze`.
- Prop de entidade nomeada genericamente (`values`) em vez do substantivo de dominio.

## Especifico por visao

- Renomear um type de metadado compartilhado sem dar grep nos dispatchers cross-feature, quando essa visao tem uma trilha de integracao adjacente que depende dele.
- Fixar um segmento de role/visao em vez de usar o param que a visao expoe pra isso.
- Usar a expressao de `canEdit` de uma visao diferente da que esta sendo trabalhada.
- Adicionar fetch novo num path legado em vez de migrar pro service V2 daquela visao.
- Conectar o container antes de confirmar que o branch do submodule realmente contem o organism, quando essa visao tiver esse gate explicito.

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/container]]
- [[Playbook/store]]
- [[Playbook/testing]]
- [[Playbook/cleanup]]
