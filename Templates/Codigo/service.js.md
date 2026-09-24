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
// services/<Area>/index.js — barrel puro, sem logica de fields/entidade aqui
export * from './<CardA>'
export * from './<CardB>'
export * from './SystemFields'

export const getOptions = async () => {
  const [ err, data ] = await request(get.Get<Area>Options)
  return [ err, { ...data?.employeeOptions } ]
}
```

```js
// services/<Area>/SystemFields.js — o unico getSystemFields, os dois eixos
import * as service from '<ROOT_ALIAS>services/SystemField'
import { <AREA>_AREA } from '@convenia/common-organisms/SystemFields/content/consts'
import { mapSystemFields, mapCustomFields } from '@convenia/employee-organisms/SystemFields/<Group>/content/mappers'

export const getSystemFields = async ({ employeeId } = {}) => {
  try {
    const [ err, sections ] = await service.getSystemFields({ employeeId, area: <AREA>_AREA.LABEL })
    return [ err, {
      systemFields: mapSystemFields(sections || []),
      customFields: mapCustomFields(sections || []),
    } ]
  } catch (err) {
    return [ err, null ]
  }
}
```

```js
// services/<Area>/<Card>.js — GET + write daquele card
import { map<Domain>Input } from '@convenia/employee-organisms/SystemFields/<Group>/content/mappers/<domain>'

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
    const { employeeId, id, schema, bypass: bcv, invalidFields = [], ...data } = params || {}
    const options = bcv ? { headers: { bcv } } : {}
    const body = map<Domain>Input({ entityId: id, schema, data, invalidFields })
    const url = `<ENTITY_BASE>`

    const { data: saved } = await rest.put(url, body, options) || {}
    return [ null, saved ]
  } catch (err) {
    return [ err, null ]
  }
}
```

Regras a nao esquecer:

- `schema` e o objeto inteiro de custom fields da sub-area (id-keyed), nunca `.schema` de dentro dele.
- Erros sempre retornam `[err, null]`, nunca throw.
- Retorno de sucesso e sempre o dado bruto (`[null, data]`), nunca `[null, true]`.
- Card single-item: `update<Entity>` e um POST create-or-replace, nome de servico `create<Entity>`.

## Referencias cruzadas

- [[Playbook/services-and-mappers]]
- [[Playbook/visao-profile]]
