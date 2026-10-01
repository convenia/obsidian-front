---
tags:
  - playbook
  - frontend
  - services
date: 2026-10-01
---

# Services

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. Transformação de shape (leitura/escrita) fica em [[Playbook/mappers]].

## O que é

**Service** é a única camada da SPA que faz requests HTTP/GraphQL. É chamado pelas actions Vuex/Pinia com params, monta a URL, chama o client `rest` e devolve uma tupla `[ err, data ]`. Service não guarda estado, não conhece componente e não decide regra de tela.

Por que isolar:

- Service fica simples e testável: só URL, verbo e tratamento de erro.
- Transformação de shape fica no mapper, fora do service — ver [[Playbook/mappers]].

## Onde fica na arquitetura

```
src/<ROOT>/
├── containers/<CONTAINER_DIR>/<Resource>.vue
├── store/<Area>/<Feature>/...
├── services/<Area>/<Feature>/
│   ├── <Resource>.js        # GET + write de um recurso
│   ├── <Metadata>.js        # fetch de metadado da feature (opcional)
│   └── index.js             # barrel puro
└── content/
    └── mappers/<entity>.js  # ver Playbook/mappers
```

Fluxo de uma escrita:

```
container (.vue) → action Store → service → map<Entity>Input → rest.<verbo>
```

## Fetch de metadado (sem híbrido)

Metadado = configuração que a API devolve pra tela montar a feature (labels, tipos, obrigatoriedade, options), separada do dado da entidade. Nem toda feature tem; quando tem:

- A feature MUST ter um único fetch de metadado, num arquivo `<Metadata>.js` com `get<Metadata>`.
- Toda tela da feature MUST ler o metadado desse fetch — nunca de uma segunda rota nem de um metadado montado à mão na SPA.
- Ao migrar, remova a busca legada. Não mantenha as duas fontes: elas divergem e a tela passa a mostrar regra diferente dependendo de qual carregou.

Forma canônica de `services/<Area>/<Feature>/<Metadata>.js`:

```js
import rest from '<REST_PATH>'
import { map<Metadata> } from '<ROOT_ALIAS>:content/mappers/<metadata>'

export const get<Metadata> = async ({ employeeId } = {}) => {
  try {
    const url = `<METADATA_ENDPOINT>`
    const { data } = await rest.get(url) || {}

    return [ null, map<Metadata>(data) ]
  } catch (err) {
    return [ err, null ]
  }
}
```

## Resource files e barrel

Prefira **uma pasta por feature com um resource file por recurso + barrel** em vez de um único `services/<Area>/index.js` com todas as funções. Arquivo único cresce sem limite, mistura recursos e vira o lugar onde mapper e lógica acabam entrando.

| Arquivo | Papel |
|---|---|
| `<Resource>.js` | resource file: GET + write (`create`/`update`/`delete`) de um recurso. Nome do arquivo = nome real do recurso no plural ou singular da rota — nunca `Card.js`. MAY agrupar sub-recursos relacionados. Importa o mapper de escrita de `<ROOT_ALIAS>:content/mappers/`. |
| `<Metadata>.js` | o único fetch de metadado (`get<Metadata>`) da feature, quando existir |
| `index.js` | barrel: só `export *` de cada arquivo. Sem lógica, sem mapper. |

```js
// services/<Area>/<Feature>/index.js
export * from './<ResourceA>'
export * from './<ResourceB>'
export * from './<Metadata>'
```

Consumidor importa pelo barrel da feature: `import * as services from '<ROOT_ALIAS>:services/<Area>/<Feature>'`.

## Nomenclatura e assinatura/funções/requisições

Prefixo da função casa com o verbo HTTP e com o nome do recurso na rota:

| Prefixo | Verbo HTTP | Uso |
|---|---|---|
| `get<Recurso>` | GET | leitura |
| `create<Recurso>` | POST | criação (ou upsert em recurso single-item) |
| `update<Recurso>` | PUT | atualização |
| `delete<Recurso>` | DELETE | remoção |

Regras:

- Nome do recurso na função MUST acompanhar o recurso na URL. Lista usa plural (`get<Entities>`); item usa singular (`create<Entity>`).
- `update*` MUST disparar PUT; `create*` MUST disparar POST. Nunca `set*`, `save*`, `fetch*`.
- Ação fora do CRUD (duplicar, bloquear, cancelar, disparar envio) MAY fugir do prefixo só quando a rota também foge do CRUD.
- GET e DELETE: destructure na assinatura (`({ employeeId, id })`).
- `create`/`update`: recebem `(params)` e destructuram no corpo, separando o resto como `data` pro mapper: `const { employeeId, id, ...data } = params || {}`.
- Query string MUST ir em `{ params: { ... } }` do client `rest` — nunca concatenada na URL. Param opcional entra por spread condicional (`...(status && { status })`), pra não mandar `?status=undefined`.
- Header opcional segue a mesma regra: `const options = token ? { headers: { token } } : {}` — nunca mandar header com `undefined`.
- REST novo MUST usar objeto. Argumento posicional é legado de GraphQL; GraphQL MUST NOT ser usado em módulo novo.

DO — `services/<Area>/<Feature>/<Entities>.js` (recurso multi-item):

```js
import rest from '<REST_PATH>'
import { map<Entity>Input } from '<ROOT_ALIAS>:content/mappers/<entity>'

export const get<Entities> = async ({ employeeId, status } = {}) => {
  try {
    const url = `<ENTITY_BASE>`
    const params = { ...(status && { status }) }

    const { data } = await rest.get(url, { params }) || {}

    return [ null, data ]
  } catch (err) {
    return [ err, null ]
  }
}

export const create<Entity> = async (params) => {
  try {
    const { employeeId, ...data } = params || {}
    const body = map<Entity>Input(data)
    const url = `<ENTITY_BASE>`

    const { data: <entity> } = await rest.post(url, body) || {}
    return [ null, <entity> ]
  } catch (err) {
    return [ err, null ]
  }
}

export const update<Entity> = async (params) => {
  try {
    const { employeeId, id, ...data } = params || {}
    const body = map<Entity>Input(data)
    const url = `<ENTITY_BASE>/${id}`

    await rest.put(url, body)
    return [ null, null ]
  } catch (err) {
    return [ err, null ]
  }
}

export const delete<Entity> = async ({ employeeId, id }) => {
  try {
    const url = `<ENTITY_BASE>/${id}`

    await rest.delete(url)

    return [ null, null ]
  } catch (err) {
    return [ err, null ]
  }
}
```

## Tratamento de erro

- Erro MUST ser capturado e retornado como `[ err, null ]` — nunca `throw`.
- Service MUST NOT transformar erro em sucesso por conta própria. Quem decide o que fazer com o erro é a store/container.
- Exceção condicional ao contrato: quando a API documenta que `404` no GET significa "recurso ainda não existe" (e não falha), o service MAY retornar valor vazio — `[ null, [] ]` em lista, `[ null, {} ]` em single-item. Sem esse contrato, `404` é erro como qualquer outro.

DO — GET com contrato "404 = sem registro":

```js
export const get<Entities> = async ({ employeeId }) => {
  try {
    const url = `<ENTITY_BASE>`
    const { data } = await rest.get(url) || {}

    return [ null, data ]
  } catch (err) {
    // contrato da API: 404 = ainda não existe <entities> cadastrado
    if (err?.response?.status === 404) return [ null, [] ] // single-item: [ null, {} ]

    return [ err, null ]
  }
}
```

DON'T — converter qualquer erro em resposta vazia:

```js
} catch (err) {
  return [ null, [] ] // 500, 403, timeout viram "lista vazia" — a tela nunca mostra o erro
}
```

## Regras do service de escrita

- POST retorna a entidade criada: `const { data: <entity> } = await rest.post(...)` → `[ null, <entity> ]`.
- PUT normalmente não devolve body → `[ null, null ]`. Só destructure a resposta quando a API tiver contrato de retorno documentado.
- DELETE retorna `[ null, null ]`.
- Nunca `[ null, true ]`.

## Recurso single-item vs multi-item

O formato do service depende de quantos registros o recurso tem por dono (ex.: por colaborador).

| | Single-item | Multi-item |
|---|---|---|
| O que é | o dono tem **no máximo um** registro do recurso | o dono tem uma **lista** de registros |
| Rota | sem `id` do item (`<ENTITY_BASE>`) | com `id` do item pra editar/remover (`<ENTITY_BASE>/${id}`) |
| Salvar | POST que cria ou substitui (upsert) | POST cria; PUT atualiza o item `id` |
| Remover | normalmente não existe | DELETE no item `id` |
| Services | `get<Entity>` + `create<Entity>` | `get<Entities>` + `create<Entity>` + `update<Entity>` + `delete<Entity>` |

- Single-item MUST NOT ter `update<Entity>`: como a rota é um upsert via POST, o nome correto é `create<Entity>`.
- Em multi-item, a store expõe uma única action de escrita e escolhe entre `create`/`update` pela presença de `id` (ver [[Playbook/store]]). O service nunca faz essa escolha.

## Anti-patterns

| Anti-pattern | Por que é ruim | Forma certa |
|---|---|---|
| Um único `services/<Area>/index.js` com todas as funções | cresce sem limite; mistura recursos | pasta por feature, resource files + barrel |
| Barrel `Main.js` ou barrel com `export * from './mappers'` | nome fora do padrão; mistura service com transformação | `index.js` só com `export *` dos resource files + `<Metadata>` |
| `` `<ENTITY_BASE>?status=${status}` `` | manda `"undefined"` quando o param não vem | `rest.get(url, { params: { ...(status && { status }) } })` |
| `update<Entity>` chamando `rest.post` | nome mente sobre o verbo; quebra busca e mock de teste | `update*` → PUT; upsert single-item → `create<Entity>` |
| `set<Entity>` decidindo POST/PUT por `!data.id` | um nome esconde dois verbos | `create<Entity>` + `update<Entity>`; quem decide é a store |
| `update<Entity>(<entity>Id, payload)` | posicional é legado GraphQL; ordem frágil | `update<Entity>({ employeeId, id, ...data })` |
| `return [ null ]` ou `return !err && response?.success` | quebra o contrato `[ err, data ]` da store | `[ null, data ]` / `[ null, null ]` / `[ err, null ]` |
| `return [ null, true ]` | `true` não é dado; consumidor não tem o que usar | DELETE/PUT sem body → `[ null, null ]` |
| `throw` dentro do service | store precisa de try/catch próprio e perde o contrato | `catch (err) { return [ err, null ] }` |
| `catch` devolvendo `[ null, [] ]` pra qualquer erro | erro real some da tela | só `404` com contrato vira vazio |

## Referências cruzadas

- [[Playbook/mappers]]
- [[Playbook/visao-profile]]
- [[Playbook/store]]
- [[Playbook/new-module]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/service.js]]
