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

A camada de service da SPA pra um card migrado: um fetch unico do metadado de campos do card, o service da area dividido por card, e onde vivem os mappers de leitura/escrita.

## O fetch unico de metadado

- O metadado de campos do card (`<FIELDS_METADATA>` — labels, tipo, obrigatoriedade, options) vem de **um** endpoint (`<METADATA_ENDPOINT>`). Nao existe split em duas rotas pra pedacos diferentes desse metadado.
- Uma chamada retorna `sections[]`; cada `section` carrega o metadado completo daquele card.

```jsonc
{
  "data": {
    "sections": [
      {
        "id": "<sectionId>",
        "fields": { "<fieldKey>": { "id": "<uuid>", "type": "select", "mandatory": true, "disabled": false, "options": [...] } }
      }
    ]
  }
}
```

> **Exemplo real:** em cards System+Custom Fields (`spa-colab/.@convenia/.../SystemFields/`) esse metadado vem em **dois eixos** — `systemFields` (nome-chaveado, campo definido pelo backend) + `customFields` (id-chaveado, campo configuravel pela empresa) — no mesmo `GET .../fields/{visao}?area=`. O eixo duplo e uma particularidade desse dominio, nao a regra geral: um card sem essa distincao usa o mesmo padrao com um unico eixo de `fields`.

## Por que a regra existe (HARD RULE — sem hibrido)

Se a area ainda usa a busca legada de metadado (uma rota por pedaco do metadado, ou um mapper que monta o metadado inline na SPA), remova — nao mantenha hibrido. A area inteira — mesmo um card irmao ainda nao migrado pro organism — le o metadado do mesmo fetch unificado.

## Area service — dividido por card

A migracao parte do `index.js` monolitico da area e fatia em `services/<Area>/{<Card>.js, Fields.js, index.js}`:

| Arquivo | Papel |
|---|---|
| `<Card>.js` | GET + write (`create`/`update`/`delete`) daquele card; importa o mapper de escrita do organism |
| `Fields.js` | o unico fetch de metadado (`get<Fields>`) da area inteira |
| `index.js` | so barrel — `export *` de cada arquivo de card + `Fields`; sem logica de metadado aqui |

## Mappers — moram no organism, nunca na SPA

- O mapper de leitura do metadado vive no organism compartilhado (`<ORGANISM_IMPORT>/content/mappers`). A SPA NAO define mapper de leitura nem faz slicing de secao inline.
- Payload de escrita tambem e do organism: o mapper de escrita daquele dominio. A SPA importa e passa os dados brutos do form + o metadado adiante.
- A SPA NAO cria `services/<Area>/mappers.js` proprio e NAO monta o payload por conta — isso duplica o mapper de escrita do organism.

## Regras do mapper de escrita

- Uma lista de campos invalidos (`invalidFields[]`) e a fonte de verdade pras chaves puladas — nunca `delete` no spread.
- Chave pulada e **omitida**, nunca setada como `null` (backend trata `null` como "limpar").

## Regras do service de escrita

- O metadado passado ao mapper de escrita e o objeto inteiro daquela sub-area, nunca so um sub-campo dele.
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
