---
tags:
  - playbook
  - frontend
  - services
date: 2026-09-24
---

# Services e Mappers

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia.

## O que e

A camada de service da SPA pra um card migrado: um fetch unico de fields, o service da area dividido por card, e onde vivem os mappers de leitura/escrita.

## O fetch unico de fields

- System fields e custom fields vem de **um** endpoint. Nao existe mais split (`/system-fields/{route}` + `/fields/{route}`).
- Uma chamada retorna `sections[]`; cada `section` carrega os dois eixos (`systemFields` por nome, `customFields` ja com chave por id).

```jsonc
{
  "data": {
    "sections": [
      {
        "id": "<sectionId>",
        "systemFields": { "<fieldName>": { "id": "<uuid>", "type": "select", "mandatory": true, "disabled": false, "options": [...] } },
        "customFields": { "<uuid>": { "label": "...", "mandatory": true, "disabled": false, "type": "select", "options": [...] } }
      }
    ]
  }
}
```

## Por que a regra existe (HARD RULE — sem hibrido)

Se a area ainda usa a canalizacao legada de system fields, remova — nao mantenha hibrido. Marcadores legados: `getEmployeeInfoSchema`, um `getCustomFields`/`mapCustomFields`/`mapSection` que mapeia na SPA, o formato `data.fields` (`{ id, schema, values }`), `mapEmployeeCustomFields`, `getAdditionalFields`, `getRequiredSystemFields`. A area inteira — mesmo um card irmao ainda nao migrado no organism — le fields do mesmo fetch unificado.

## Area service — dividido por card

A migracao parte do `index.js` monolitico da area e fatia em `services/<Area>/{<Card>.js, SystemFields.js, index.js}`:

| Arquivo | Papel |
|---|---|
| `<Card>.js` | GET + write (`create`/`update`/`delete`) daquele card; importa `map<Domain>Input` do organism |
| `SystemFields.js` | o unico `getSystemFields` (os dois eixos) da area inteira |
| `index.js` | so barrel — `export *` de cada arquivo de card + `SystemFields`; sem logica de fields aqui |

## Mappers — moram no organism, nunca na SPA

- `mapSystemFields`/`mapCustomFields` vivem em `@convenia/employee-organisms/SystemFields/<Group>/content/mappers`. A SPA NAO define mapper de leitura nem faz `pickSection` inline.
- Payload de escrita tambem e do organism: `map<Domain>Input`. A SPA importa e passa os dados brutos do form + schema id-keyed adiante.
- A SPA NAO cria `services/<Area>/mappers.js` e NAO chama `buildPayloadWithCustomFields` diretamente — isso duplica o mapper de escrita.

## Regras do write mapper

- `invalidFields[]` e a fonte de verdade pras chaves puladas — nunca `delete` no spread.
- Chave pulada e **omitida**, nunca setada como `null` (backend trata `null` como "limpar").

## Regras do service de escrita

- `schema` e o objeto inteiro de custom fields da sub-area (id-keyed), nao `customFields[sub].schema`.
- Sempre desestruturar `invalidFields = []` com default.
- Erros sao capturados e retornados como `[err, null]` — nunca lancados (throw).
- Retornar o registro criado/atualizado bruto (`[null, data]`) — nunca `[null, true]`.

## Single-item vs multi-item

- **Card single-item**: o "update" e um POST create-or-replace (upsert). Nome do service: `create<Entity>`. Registro ausente (404) e tratado como registro vazio, nao como erro: `if (err?.response?.status === 404) return [null, null]`.
- **Card multi-item**: services `create`/`update`/`delete` separados na camada de service — a store ainda expoe uma unica action de escrita (ver [[Playbook/store]]).

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/store]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/service.js]]
