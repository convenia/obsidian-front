---
tags:
  - template
  - frontend
  - services
date: 2026-09-24
---

# Template: service.js

Esqueleto da camada de service da area, dividido por recurso. Guia: [[Playbook/services-and-mappers]]. Tokens: [[Playbook/visao-profile]].

```js
// services/<Area>/index.js — barrel puro, sem logica de metadado/entidade aqui
export * from './<FeatureA>'
export * from './<FeatureB>'
export * from './<FeatureC>'
```

```js
// services/<Area>/<FeatureA>.js — nome real = o recurso/dominio (ex.: Personal.js, Dependents.js)
// nome da funcao = prefixo (get/create/update/delete) + recurso, casado com o verbo HTTP e proximo do nome da rota
import rest from '<REST_MW>'
import { mapEntityInput } from '<ROOT_ALIAS>content/mappers/<entity>'

export const get<Entity> = async ({ employeeId } = {}) => {
  try {
    const url = `<ENTITY_BASE>` // ja resolvido pra visao (admin/supervisor/colab) — ver [[Playbook/visao-profile]], nao remonte o path na mao
    const { data } = await rest.get(url)

    return [ null, data ]
  } catch (err) {
    return [ err, null ]
  }
}

export const create<Entity> = async ({ employeeId, ...data } = {}) => {
  try {
    // mapper so entra se o shape do form nao bate com o esperado pela API
    const body = mapEntityInput({ ...data })
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
    const body = mapEntityInput({ entityId: id, ...data })
    const url = `<ENTITY_BASE>/${id}`

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
// variante sem mapper — quando o payload do form ja bate com o shape esperado pela API
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

Regras a nao esquecer:

- `<REST_MW>` (import do client REST) varia por SPA — resolva o token pela [[Playbook/visao-profile]], nunca hardcode um path de import.
- `<ENTITY_BASE>` ja vem resolvido pra visao certa (o segmento pode ser `admin`, `supervisor`, `employee`, ou nem existir, dependendo da visao) — nunca remonte esse path na mao concatenando `<VISAO>` direto na URL do service.
- Prefixo da funcao sempre `get`/`create`/`update`/`delete`, casado com o verbo HTTP (`GET`/`POST`/`PUT`/`DELETE`) e com o nome do recurso na rota — nunca um nome generico tipo `fetch`/`save`/`remove`.
- Acoes que nao sao CRUD puro (duplicar, bloquear/desbloquear, cancelar, disparar um envio) podem fugir desse prefixo — mas so quando a rota em si tambem foge do CRUD. E excecao pontual, nunca o padrao pra um recurso novo. Se o nome for `update*`/`create*`/etc., o verbo HTTP por baixo tem que bater — nunca um `updateX` que na real dispara um POST.
- Destructure dos parametros direto na assinatura da funcao (`({ employeeId, ... })`). So use `params` + destructure interno quando a lista de campos ficar longa (ex.: um `update` com varios campos passa de ~100 caracteres numa linha so).
- Servicos que ainda falam com GraphQL usam argumento posicional (ex.: `updateGoal(goalId, payload)`) em vez de objeto desestruturado — isso e **codigo legado**. GraphQL nao deve ser usado em rotas/modulos novos; REST novo sempre desestrutura objeto.
- Mapper de leitura segue o nome `mapEntity`; mapper de escrita segue `mapEntityInput` — o nome acompanha a entidade lida/escrita, nao um "domain" generico solto.
- O metadado passado ao mapper de escrita e o objeto inteiro daquela sub-area, nunca um sub-campo dele.
- Mapper de escrita (`content/mappers`) so e criado/importado quando o payload do form precisa de transformacao. Se o dado ja bate com o shape esperado pela API, o service manda direto pro `rest.<verbo>`, sem mapper.
- `create<Entity>` nao usa `invalidFields` — esse conceito e so de update parcial (campo que pode ser pulado numa edicao), nao existe em criacao.
- `update<Entity>`/`delete<Entity>`: PUT e DELETE normalmente nao devolvem body — nao destruture a resposta do `rest.put`/`rest.delete`, retorne o dado local ja conhecido. `create<Entity>` (POST) e diferente: normalmente devolve e destructura o body da resposta.
- Erros sempre retornam `[err, null]`, nunca throw.
- Retorno de sucesso e sempre o dado bruto (`[null, data]`), nunca `[null, true]`.
- Recurso single-item: `update<Entity>` e um POST create-or-replace, nome de servico `create<Entity>`.
- Recurso multi-item: `delete<Entity>` recebe o `id` do item e chama `rest.delete` no recurso individual.

## Referencias cruzadas

- [[Playbook/services-and-mappers]]
- [[Playbook/visao-profile]]
