---
tags:
  - template
  - frontend
  - mappers
date: 2026-10-01
---

# Template: mapper.js

Boilerplate do par de mappers de leitura/escrita de uma entidade. Guia: [[Playbook/mappers]]. Tokens: [[Playbook/visao-profile]].

```js
// <ROOT_ALIAS>:content/mappers/<entity>.js — um arquivo por entidade
import { camelToSnakeKeys, formatDateToString } from '@convenia/helpers'

// leitura: API → aplicação
export const map<Entity> = (data = {}) => ({
  ...data,
  <relation>Id: data?.<relation>?.id,
  startDate: formatDateToString(data?.startDate),
})

// escrita: aplicação → body da API
export const map<Entity>Input = (data = {}) => camelToSnakeKeys({
  <relation>Id: data?.<relation>Id,
  startDate: formatDateToString(data?.startDate),
  ...(data?.<field> && { <field>: data.<field> }),
})
```

```js
// variante só de leitura — quando o dado já bate com o body da API (sem map<Entity>Input)
export const map<Entity> = (data = {}) => ({
  ...data,
  <relation>Id: data?.<relation>?.id,
})
```

Regras a não esquecer:

- Arquivo mora em `<ROOT_ALIAS>:content/mappers/<entity>.js`. Se um pacote compartilhado já exporta o mapper da entidade, importe dele — não crie este arquivo.
- Nome acompanha a entidade: leitura `map<Entity>`, escrita `map<Entity>Input` — nunca um nome genérico de "domain".
- Dado como primeiro argumento, com default (`(data = {})`). Contexto extra MAY vir num segundo argumento objeto.
- Mapper é função pura: sem `rest`, sem store, sem efeito colateral.
- Campo opcional por spread condicional, nunca `delete` no objeto.
- `map<Entity>Input` só existe quando há transformação. Sem transformação, o service manda o dado direto pro `rest.<verbo>`.
- Mapper MUST NOT ser definido dentro de arquivo de service nem em `services/<Area>/mappers.js`.
- Teste do mapper fica em `content/mappers/tests/`.

## Referências cruzadas

- [[Playbook/mappers]]
- [[Playbook/services]]
- [[Templates/Codigo/service.js]]
- [[Playbook/visao-profile]]
