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

A camada de service da SPA: um fetch unico do metadado de campos, o service da area dividido por recurso/feature, e onde vivem os mappers de leitura/escrita.

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

## Por que a regra existe (HARD RULE — sem hibrido)

Se a area ainda usa a busca legada de metadado (uma rota por pedaco do metadado, ou um mapper que monta o metadado inline na SPA), remova — nao mantenha hibrido. A area inteira — mesmo um card irmao ainda nao migrado — le o metadado do mesmo fetch unificado.

## Area service — dividido por recurso

A migracao parte do `index.js` monolitico da area e fatia em `services/<Area>/{<Feature>.js, Fields.js, index.js}`:

| Arquivo | Papel |
|---|---|
| `<Feature>.js` | GET + write (`create`/`update`/`delete`) daquele recurso; nome do arquivo e o nome real do recurso/dominio (ex.: `Personal.js`, `Dependents.js`) — nunca `Card.js`. Um arquivo pode agrupar mais de um sub-recurso relacionado quando fizer sentido (ex.: varios tipos de documento no mesmo arquivo). Importa o mapper de escrita de `content/mappers/` do modulo. |
| `Fields.js` | o unico fetch de metadado (`get<Fields>`) da area inteira |
| `index.js` | so barrel — `export *` de cada arquivo de recurso + `Fields`; sem logica de metadado aqui |

## Nomenclatura — prefixo casado com o verbo HTTP

Toda funcao de service segue o mesmo prefixo, casado com o verbo HTTP e com o nome do recurso na rota:

| Prefixo | Verbo HTTP | Uso |
|---|---|---|
| `get<Recurso>` | GET | leitura |
| `create<Recurso>` | POST | criacao (ou upsert em recurso single-item) |
| `update<Recurso>` | PUT | atualizacao |
| `delete<Recurso>` | DELETE | remocao |

O nome do recurso na funcao acompanha o nome do recurso na rota — nao invente um nome de funcao desalinhado da URL que ela chama.

Acoes que nao sao CRUD puro (duplicar, bloquear/desbloquear, cancelar, disparar um envio) podem fugir desse prefixo — mas so quando a rota em si tambem foge do CRUD. E excecao pontual, nunca o padrao pra um recurso novo. Se o nome for `update*`/`create*`/etc., o verbo HTTP por baixo tem que bater — nunca um `updateX` que na real dispara um POST.

Parametros: destructure direto na assinatura da funcao (`({ employeeId, ... })`). So use `params` + destructure interno quando a lista de campos ficar longa (ex.: um `update` com varios campos passa de ~100 caracteres numa linha so).

Servicos que ainda falam com GraphQL usam argumento posicional (ex.: `updateGoal(goalId, payload)`) em vez de objeto desestruturado — isso e **codigo legado**. GraphQL nao deve ser usado em rotas/modulos novos; REST novo sempre desestrutura objeto.

## Mappers — arquivo dedicado em content/mappers, nunca inline no service

- Mapper de leitura (`mapEntity`) e de escrita (`mapEntityInput`) moram **dentro do proprio modulo** que consome o service, em `<ROOT_ALIAS>content/mappers/<entity>` — irmao de `content/consts/` e `content/forms/` (ver [[Playbook/new-module]]). Nao e um pacote externo compartilhado; e codigo do proprio modulo.
- Nome do mapper acompanha a entidade: leitura e `mapEntity`, escrita e `mapEntityInput` — nunca um nome de "domain" generico solto.
- Mapper de escrita so e criado quando o payload do form precisa de transformacao antes de virar body da API. Se o dado do form ja bate com o shape esperado pela API, o service chama `rest.<verbo>` direto, sem mapper.
- A regra que importa e a de **separacao de arquivo**: o mapper nunca e definido inline dentro do arquivo de service (`<Feature>.js`) — sempre um arquivo proprio em `content/mappers/`, importado pelo service. Definir a funcao de mapper direto dentro do `services/index.js` (misturando fetch + transformacao no mesmo arquivo) e o red flag a evitar, nao o fato de o mapper existir dentro do modulo.
- O service importa o mapper e passa os dados brutos do form + o metadado adiante — a logica de transformacao fica isolada em `content/mappers/`, nunca duplicada ad-hoc dentro do service.


## Regras do service de escrita

- O metadado passado ao mapper de escrita e o objeto inteiro daquela sub-area, nunca so um sub-campo dele.
- Erros sao capturados e retornados como `[err, null]` — nunca lancados (throw).
- Retornar o registro criado/atualizado bruto (`[null, data]`) — nunca `[null, true]`.
- PUT e DELETE normalmente nao devolvem body na resposta — `update<Entity>`/`delete<Entity>` nao devem depender de destructure da resposta do `rest.put`/`rest.delete`; retorne o dado local ja conhecido. `create<Entity>` (POST) e diferente: normalmente devolve e destructura o body da resposta.

## Recurso single-item vs multi-item

- **Recurso single-item**: o "update" e um POST create-or-replace (upsert). Nome do service: `create<Entity>`.
- **Recurso multi-item**: services `create`/`update`/`delete` separados na camada de service, cada um recebendo o `id` do item pra `update<Entity>`/`delete<Entity>` — a store ainda expoe uma unica action de escrita (ver [[Playbook/store]]).

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/store]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/service.js]]
