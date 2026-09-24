---
tags:
  - playbook
  - frontend
  - container
date: 2026-09-24
---

# Card Container

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O shape do container e identico entre visoes e entre cards — so os valores mudam. Convencoes de props/atributos do proprio organism ficam em [[Playbook/component-conventions]].

## O que e

O container fino do card: renderiza um organism, resolve o callback `@submit`/`@remove`, e possui o carregamento de dados/options. Estado de modal, schema de form e montagem de payload ficam no organism, nao aqui.

## `canEdit` — o unico valor comportamental que muda

`<CAN_EDIT_EXPR>` resolve por visao:

- visao admin/gestor: tipicamente `!isDisconnected.value` ou equivalente flag de permissao.
- visao self-service: tipicamente fixo `true` nos cards V2 migrados — sem config de store.

Passe `:can-edit="canEdit"` pro organism, que possui o esconder do botao de editar. Nao use `v-if` no template do container pra esse gate — o organism decide isso, senao cada visao copia o gate.

## Naming — alinhar toda superficie ao organism

Antes de nomear qualquer coisa, leia o fragment do organism — ele e a fonte de verdade do token de dominio. Quando a SPA e o organism divergem, **o organism vence**: renomeie o lado da SPA. Nunca invente uma variante local de nome.

| Superficie | Nome derivado do organism |
|---|---|
| Arquivo do container | `<CONTAINER_DIR><Card>.vue` |
| `name` do container | `<Module><Card>Container` (PascalCase, sem sufixo `Section`/`Info`) |
| Tag no template | `<entity>-card`, nunca `card-<entity>` |
| Arquivo/funcoes de service | `services/<Area>/<Card>.js`; `get<Entity>`/`create<Entity>`/`update<Entity>`/`delete<Entity>` |
| Slice + types de store | slice `<Sub>`; `<TYPE_PREFIX>_<VERB>_<AREA>_<ENTITY>` |

## Props que o container passa pro organism

O container passa a entidade e o metadado de campos que o organism precisa pra renderizar. O numero e o shape exato do metadado (`<FIELDS_METADATA>`) variam por dominio — ver [[Playbook/services-and-mappers]] pra como o metadado chega na store.

## Convencoes

- `<script setup>` e obrigatorio pra container novo.
- `mapState`/`mapActions` vem de `@convenia/macros/vuex-composition.macro` — nunca de `vuex` direto, nunca via `this.$store`.
- `useRoute` vem de `vue-composition-wrapper`, nao de `vue-router`.
- Atributos do template (inclusive na tag do organism) seguem a ordenacao alfabetica de [[Playbook/component-conventions]]: diretivas estruturais → atributos alfabeticos → eventos alfabeticos.
- O organism le contexto via `inject` (ver [[Playbook/loader]] pra onde colocar o `provide`).

## Capacidades de attachment (so cards com anexo)

Cards **com** attachment passam props de capacidade derivadas do metadado; cards **sem** attachment omitem (o organism remove o schema de arquivo quando ausentes). O nome exato dessas props e definido pelo organism — confirme contra o fragment antes de fixar a derivacao.

## Contrato de evento

O organism possui o estado do modal + feedback; o container so resolve o callback e possui o loading de dados/options. Nao existe evento `@edit` na maioria dos casos — o organism abre seu proprio editor.

**`@submit`** — `{ id, data, addedFiles, deletedFiles, callback }`:

1. Despacha a **unica** action `update<Entity>` com `{ id, ...data, employeeId }` → `[err, entity]`. Passe `id` direto — nunca decida create vs update no container, a store decide via `!payload.id`.
2. Erro → `callback(err)` e retorna (modal fica aberto com feedback).
3. Se `addedFiles`/`deletedFiles`, chama `update<Entity>Files` uma vez com `{ upload, remove }`.
4. `callback()` sem args — organism fecha o modal.
5. Refaz o fetch da entidade.

**`@remove`** — `{ id, callback }`: chama `delete<Entity>({ id, employeeId })` e `callback(err)`.

## O que o container NAO deve fazer

- Montar schema de form ou form primitive de modal.
- Guardar estado de modal aberto/fechado, edit/create, ou flags de arquivo removido.
- Decidir create vs update.
- Montar payload — quem monta e o mapper de escrita do organism.
- Buscar/passar options pra um select cujas options ja vem no metadado.
- Renderizar varios cards via um `cards` computed — o organism itera as entidades internamente.

Se algum item acima vazar pro container, a migracao esta incompleta.

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/loader]]
- [[Playbook/store]]
- [[Playbook/component-conventions]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/container.vue]]
