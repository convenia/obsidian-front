---
tags:
  - playbook
  - frontend
  - vuex
date: 2026-10-01
---

# Store Types

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. Uso do slice (state, mutations, actions, getters) fica em [[Playbook/store]].

## O que é

**Type** é a constante string que nomeia uma action, mutation ou getter Vuex. O container e a store nunca escrevem o nome na mão: sempre importam a constante (`types.<NOME>`).

## Onde fica

- Todo módulo MUST ter um único `src/<ROOT>/store/types.js` com todas as constantes. NÃO criar um `types.js` por feature.
- `types.js` MUST começar com `export * from '@types'`. Isso re-exporta os types comuns da SPA (ex.: `FEEDBACKS_ADD`).

```
src/<ROOT>/store/
├── types.js          # todos os types do módulo
└── <Feature>/index.js
```

## Por que usar

- Os slices não usam `namespaced: true`, então todo nome é global. O valor do type é o que evita colisão entre módulos.
- Um nome previsível deixa achar no grep quem busca, quem escreve e quem deriva um dado.
- Container e store dependem da mesma constante: renomear num lugar só quebra no import, não em runtime silencioso.

## Como usar

### Fórmula do nome

```
<TYPE_PREFIX>_<FEATURE>_<VERB>_<AREA>
```

| Parte | Papel | O que é |
|---|---|---|
| `<TYPE_PREFIX>` | dono | módulo em `UPPER_SNAKE` (valor da [[Playbook/visao-profile]]) |
| `<FEATURE>` | contexto | container/feature em `UPPER_SNAKE`, o mesmo nome da pasta `store/<Feature>/` |
| `<VERB>` | ação | o que o type faz (ver "Verbos") |
| `<AREA>` | alvo | recurso ou sub-área. Singular pra item (`<AREA>`), plural pra lista (`<AREAS>`) |

- Todas as quatro partes MUST existir, nessa ordem.
- `<AREA>` MUST usar o mesmo substantivo do service e do mapper (ver [[Playbook/services]]).

### Exemplo de leitura

```
EMPLOYEES_DETAILS_GET_PERSONAL
```

| Parte | Valor | Leitura |
|---|---|---|
| `<TYPE_PREFIX>` | `EMPLOYEES` | módulo de colaboradores |
| `<FEATURE>` | `DETAILS` | feature/container de detalhes |
| `<VERB>` | `GET` | action que busca no service |
| `<AREA>` | `PERSONAL` | sub-área de dados pessoais |

Os outros types da mesma área mudam só o verbo:

- `EMPLOYEES_DETAILS_SET_PERSONAL` — mutation que escreve o state.
- `EMPLOYEES_DETAILS_UPDATE_PERSONAL` — action que cria ou atualiza.

### Valor

- Valor MUST ser `'<TYPE_NS>' + nome da constante`.
- `<TYPE_NS>` MUST ser um só pro módulo inteiro.

```js
export const <TYPE_PREFIX>_<FEATURE>_GET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_GET_<AREAS>'
```

### Verbos

Cada verbo tem um papel só. Action e mutation MUST ter types distintos — nunca a mesma constant nos dois.

| Verbo | Papel | Faz | Forma |
|---|---|---|---|
| `GET` | action | busca no service e faz `commit` do `SET` | `<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>` |
| `UPDATE` | action | cria ou atualiza (decide por `!payload.id`) | `<TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>` |
| `DELETE` | action | remove | `<TYPE_PREFIX>_<FEATURE>_DELETE_<AREA>` |
| `SET` | mutation | escreve no state | `<TYPE_PREFIX>_<FEATURE>_SET_<AREAS>` |
| `RESET` | mutation | volta a chave ao valor vazio | `<TYPE_PREFIX>_<FEATURE>_RESET_<AREAS>` |
| `HAS` | getter | boolean derivado | `<TYPE_PREFIX>_<FEATURE>_HAS_<AREAS>` |
| `CAN_<AÇÃO>` | getter | boolean de permissão/estado derivado | `<TYPE_PREFIX>_<FEATURE>_CAN_EDIT_<AREA>` |

- NÃO existe `CREATE` quando o recurso tem create e update: os dois são a mesma action `UPDATE` (ver [[Playbook/store]]).
- Ação fora do CRUD (duplicar, compartilhar, arquivar, filtrar) MAY usar o verbo do domínio (`<TYPE_PREFIX>_<FEATURE>_SHARE_<AREA>`), sempre como action.

### Agrupamento e ordem

- Types MUST ficar agrupados por feature, com um comentário de seção por grupo.
- Dentro do grupo, ordem alfabética estrita pelo nome da constante.
- Ao tocar um grupo, reordene o grupo inteiro — inclusive entradas antigas. Nunca só anexe no final.

```js
// store/types.js
export * from '@types'

// <Feature>
export const <TYPE_PREFIX>_<FEATURE>_DELETE_<AREA> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_DELETE_<AREA>'
export const <TYPE_PREFIX>_<FEATURE>_GET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_GET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_HAS_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_HAS_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_RESET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_RESET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_SET_<AREAS> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_SET_<AREAS>'
export const <TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA> = '<TYPE_NS><TYPE_PREFIX>_<FEATURE>_UPDATE_<AREA>'
```

### Import

- Quando o arquivo usa só types do módulo, importe um só, como `types`. O `types.js` do módulo já re-exporta os types comuns.
- Quando o arquivo precisa dos types comuns da SPA **e** dos types do módulo separados, o comum MUST se chamar `types` e o do módulo MUST se chamar `types<Area>`.

```js
// só types do módulo
import * as types from '<ROOT_ALIAS>:types'
```

```js
// types comuns + types do módulo
import * as types from '@types'
import * as types<Area> from '<ROOT_ALIAS>:types'
```

### Rename de type legado

Ao tocar uma área com type fora da convenção, renomeie na mesma mudança:

1. Grep do nome antigo no repo inteiro (store, containers, composables, testes, outros módulos).
2. Renomeie constante e valor juntos.
3. Atualize todo dispatcher, `mapActions`/`mapMutations`/`mapGetters` e teste encontrado.

## Anti-patterns

| Anti-pattern | Por que é ruim | Forma certa |
|---|---|---|
| `<FEATURE>_GET_<AREAS>` (sem `<TYPE_PREFIX>`) | colide com outro módulo; não diz de quem é | `<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>` |
| `<TYPE_PREFIX>_GET_<AREAS>` (sem `<FEATURE>`) | não diz qual feature do módulo usa o type | `<TYPE_PREFIX>_<FEATURE>_GET_<AREAS>` |
| `<TYPE_PREFIX>_<FEATURE>_<AREAS>_GET` / `GET_<TYPE_PREFIX>_...` | verbo fora da posição; quebra a busca | verbo depois de `<FEATURE>` |
| Mesma constant como action **e** mutation | não dá pra saber quem busca e quem escreve | action `GET`, mutation `SET` |
| Dois namespaces no mesmo módulo | valor do type deixa de identificar o módulo | um `<TYPE_NS>` por módulo |
| Valor diferente do nome | grep não acha | valor = `<TYPE_NS>` + nome |
| `CREATE` + `UPDATE` pro mesmo recurso | container escolhe o branch | só `UPDATE` |
| Constante nova anexada no fim do arquivo | ordem quebra; duplicata passa despercebida | inserir no grupo em ordem alfabética |
| Alias de import livre (`etypes`, `commonTypes`, `moduleTypes`) | cada arquivo chama de um jeito | `types` (comum) + `types<Area>` (módulo) |

## Referências cruzadas

- [[Playbook/store]]
- [[Playbook/visao-profile]]
- [[Playbook/services]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/store-module.js]]
