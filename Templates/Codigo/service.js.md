---
tags:
  - template
  - frontend
  - services
date: 2026-09-24
---

# Template: service.js

Esqueleto da camada de service da area, dividido por card. Guia: [[Playbook/services-and-mappers]]. Tokens: [[Playbook/visao-profile]].

```js
// services/<Area>/index.js — barrel puro, sem logica de metadado/entidade aqui
export * from './<CardA>'
export * from './<CardB>'
export * from './Fields'

export const getOptions = async () => {
  const [ err, data ] = await request(get.Get<Area>Options)
  return [ err, { ...data?.employeeOptions } ]
}
```

```js
// services/<Area>/Fields.js — o unico fetch de metadado do card
import * as service from '<ROOT_ALIAS>services/Fields'
import { <AREA_CONST> } from '<ORGANISM_IMPORT>/content/consts'
import { mapFieldsMetadata } from '<ORGANISM_IMPORT>/content/mappers'

export const getFieldsMetadata = async ({ employeeId } = {}) => {
  try {
    const [ err, sections ] = await service.getFields({ employeeId, area: <AREA_CONST>.LABEL })
    return [ err, mapFieldsMetadata(sections || []) ]
  } catch (err) {
    return [ err, null ]
  }
}
```

```js
// services/<Area>/<Card>.js — GET + write daquele card
import { map<Domain>Input } from '<ORGANISM_IMPORT>/content/mappers/<domain>'

export const get<Entity> = async (params) => {
  try {
    const { employeeId } = params || {}
    const url = `<ENTITY_BASE>`
    const { data } = await rest.get(url)
    return [ null, data ]
  } catch (err) {
    if (err?.response?.status === 404) return [ null, null ]   // registro ausente != erro
    return [ err, null ]
  }
}

export const update<Entity> = async (params) => {
  try {
    const { employeeId, id, metadata, bypass: bcv, invalidFields = [], ...data } = params || {}
    const options = bcv ? { headers: { bcv } } : {}
    const body = map<Domain>Input({ entityId: id, metadata, data, invalidFields })
    const url = `<ENTITY_BASE>`

    const { data: saved } = await rest.put(url, body, options) || {}
    return [ null, saved ]
  } catch (err) {
    return [ err, null ]
  }
}
```

> **Exemplo real:** em cards System+Custom Fields `Fields.js` chama `mapSystemFields`/`mapCustomFields` (dois mappers, um por eixo) em vez de um `mapFieldsMetadata` unico, e `update<Entity>` recebe `schema` (o objeto id-keyed de custom fields) em vez de um `metadata` generico.

Regras a nao esquecer:

- O metadado passado ao mapper de escrita e o objeto inteiro daquela sub-area, nunca um sub-campo dele.
- Erros sempre retornam `[err, null]`, nunca throw.
- Retorno de sucesso e sempre o dado bruto (`[null, data]`), nunca `[null, true]`.
- Card single-item: `update<Entity>` e um POST create-or-replace, nome de servico `create<Entity>`.

## Referencias cruzadas

- [[Playbook/services-and-mappers]]
- [[Playbook/visao-profile]]
