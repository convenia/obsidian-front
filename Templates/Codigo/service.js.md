---
tags:
  - template
  - frontend
  - services
date: 2026-10-01
---

# Template: service.js

Boilerplate da camada de service da área, dividido por recurso. Guia: [[Playbook/services]]. Tokens: [[Playbook/visao-profile]].

```js
// services/<Area>/index.js — barrel puro, sem lógica de metadado/entidade aqui
export * from './<FeatureA>'
export * from './<FeatureB>'
export * from './<FeatureC>'
```

```js
// services/<Area>/<FeatureA>.js — nome real = o recurso/domínio (ex.: Personal.js, Dependents.js)
// nome da função = prefixo (get/create/update/delete) + recurso, casado com o verbo HTTP e próximo do nome da rota
import rest from '<REST_PATH>'
// mapper mora no módulo; só importe do organism quando ele já exporta o mapper da entidade
import { map<Entity>Input } from '<ROOT_ALIAS>:content/mappers/<entity>'

export const get<Entity> = async ({ employeeId } = {}) => {
  try {
    const url = `<ENTITY_BASE>` // já resolvido pra visão (admin/supervisor/colab) — ver [[Playbook/visao-profile]], não remonte o path na mão
    const { data } = await rest.get(url) || {}

    return [ null, data ]
  } catch (err) {
    // só com contrato da API "404 = sem registro"; sem contrato, remova esta linha
    if (err?.response?.status === 404) return [ null, [] ] // single-item: [ null, {} ]

    return [ err, null ]
  }
}

export const create<Entity> = async (params) => {
  try {
    const { employeeId, ...data } = params || {}
    // mapper só entra se o shape do form não bate com o esperado pela API
    const body = map<Entity>Input(data)
    const url = `<ENTITY_BASE>`

    const { data: created } = await rest.post(url, body) || {}

    return [ null, created ]
  } catch (err) {
    return [ err, null ]
  }
}

export const update<Entity> = async (params) => {
  try {
    const { employeeId, id, ...data } = params || {}
    const body = map<Entity>Input(data)
    const url = `<ENTITY_BASE>/${id}`

    // PUT sem contrato de retorno: não destructure a resposta
    await rest.put(url, body)
    return [ null, null ]
  } catch (err) {
    return [ err, null ]
  }
}

export const delete<Entity> = async ({ employeeId, id } = {}) => {
  try {
    const url = `<ENTITY_BASE>/${id}`

    await rest.delete(url)

    return [ null, null ]
  } catch (err) {
    return [ err, null ]
  }
}
```

```js
// variante sem mapper — quando o payload do form já bate com o shape esperado pela API
export const update<Entity> = async (params) => {
  try {
    const { employeeId, id, ...data } = params || {}
    const url = `<ENTITY_BASE>/${id}`

    await rest.put(url, data)
    return [ null, null ]
  } catch (err) {
    return [ err, null ]
  }
}
```

Regras a não esquecer:

- `<REST_PATH>` (import do client REST) varia por SPA — resolva o token pela [[Playbook/visao-profile]], nunca hardcode um path de import.
- `<ENTITY_BASE>` já vem resolvido pra visão certa (o segmento pode ser `admin`, `supervisor`, `employee`, ou nem existir, dependendo da visão) — nunca remonte esse path na mão concatenando `<VISAO>` direto na URL do service.
- Prefixo da função sempre `get`/`create`/`update`/`delete`, casado com o verbo HTTP (`GET`/`POST`/`PUT`/`DELETE`) e com o nome do recurso na rota — nunca um nome genérico tipo `fetch`/`save`/`remove`.
- Ações que não são CRUD puro (duplicar, bloquear/desbloquear, cancelar, disparar um envio) podem fugir desse prefixo — mas só quando a rota em si também foge do CRUD. É exceção pontual, nunca o padrão pra um recurso novo. Se o nome for `update*`/`create*`/etc., o verbo HTTP por baixo tem que bater — nunca um `updateX` que na real dispara um POST.
- Query string vai em `rest.get(url, { params })`, com param opcional por spread condicional (`...(area && { area })`) — nunca concatenada na URL.
- GET/DELETE: destructure na assinatura (`({ employeeId, id })`). `create`/`update`: recebem `(params)` e destructuram no corpo (`const { employeeId, id, ...data } = params || {}`), passando `data` ao mapper.
- 404 → valor vazio (`[ null, [] ]` lista, `[ null, {} ]` single-item) **só** quando o contrato da API define 404 como "sem registro". Sem contrato, 404 é erro: `[ err, null ]`. Service nunca transforma erro em sucesso por conta própria.
- Serviços que ainda falam com GraphQL usam argumento posicional (ex.: `updateGoal(goalId, payload)`) em vez de objeto desestruturado — isso é **código legado**. GraphQL não deve ser usado em rotas/módulos novos; REST novo sempre desestrutura objeto.
- Mapper de escrita (`<ROOT_ALIAS>:content/mappers/<entity>`) só é importado quando o payload do form precisa de transformação; senão o service manda direto pro `rest.<verbo>`. Regras de nome e assinatura: [[Templates/Codigo/mapper.js]].
- `update<Entity>`/`delete<Entity>`: PUT e DELETE normalmente não devolvem body → `[ null, null ]`. Só destructure a resposta do PUT quando a API tiver contrato de retorno. `create<Entity>` (POST) destructura e retorna a entidade criada.
- Erros sempre retornam `[err, null]`, nunca throw.
- Retorno de sucesso é sempre o dado bruto (`[null, data]`), nunca `[null, true]`.
- Recurso single-item: `update<Entity>` é um POST create-or-replace, nome de serviço `create<Entity>`.
- Recurso multi-item: `delete<Entity>` recebe o `id` do item e chama `rest.delete` no recurso individual.

## Referências cruzadas

- [[Playbook/services]]
- [[Playbook/mappers]]
- [[Templates/Codigo/mapper.js]]
- [[Playbook/visao-profile]]
