---
tags:
  - playbook
  - frontend
  - mappers
date: 2026-10-01
---

# Mappers

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. Quem chama o mapper de escrita é o service — ver [[Playbook/services]].

## O que é

**Mapper** é uma função pura que trata os dados, traduzindo o shape de um formato para outro. Existem dois tipos:

- Mapper de leitura (`map<Entity>`): resposta da API → shape que a aplicação consome (store, form, tabela, componente).
- Mapper de escrita (`map<Entity>Input`): dado da aplicação → body que a API espera (ex.: `camelToSnakeKeys`, formatação de data, id de relação achatado).

## Onde fica

- Um arquivo por entidade em `<ROOT_ALIAS>:content/mappers/<entity>.js`, no mesmo nível de `content/forms/` e `content/consts/` (ver [[Playbook/new-module]]).
- Mapper de leitura é chamado por quem consome o dado (store ou componente), não dentro do service.
- Mapper de escrita é chamado pelo service, antes do `rest.<verbo>`.
- Exceção: quando um pacote compartilhado (ex.: organism de `@convenia/*`) **já exporta** o mapper daquela entidade, importe dele — não duplique no módulo.

## Por que usar

- O shape da API (snake_case, objetos de relação aninhados, datas em string) é diferente do shape da aplicação (camelCase, ids planos, datas formatadas).
- Sem mapper, cada service/store faz essa conversão à mão e as cópias divergem.
- Mudou o contrato da API, muda um arquivo só.
- Função pura: testável isolada, sem mock de HTTP. Teste fica em `content/mappers/tests/`.

## Como usar

- Arquivo traz o par `map<Entity>` + `map<Entity>Input`.
- Nome acompanha a entidade (`map<Entity>`, `map<Entity>Input`) — nunca um nome genérico de "domain".
- Assinatura: os dois recebem o dado como primeiro argumento, com default (`(data = {})`). Se precisar de contexto extra, MAY receber um segundo argumento objeto (`(data = {}, { <option> } = {})`).
- Campo opcional entra por spread condicional (`...(data?.<field> && { <field>: data.<field> })`) — nunca `delete` no objeto.
- Service importa o mapper pelo path completo do arquivo e passa os dados brutos.
- Mapper de escrita só existe quando há transformação. Se o dado já bate com o body da API, o service chama `rest.<verbo>` direto.
- Mapper MUST NOT ser definido dentro de arquivo de service nem em `services/<Area>/mappers.js`.
- Mapper MUST NOT chamar `rest`, store ou ter efeito colateral.

DO — `<ROOT_ALIAS>:content/mappers/<entity>.js`:

```js
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
  ...(data?.observation && { observation: data.observation }),
})
```

## Anti-patterns

| Anti-pattern | Por que é ruim | Forma certa |
|---|---|---|
| `// Mappers` + `export const map<Entity>` dentro de `services/<Area>/index.js` | mistura fetch com transformação; mapper não é testável isolado | arquivo próprio em `<ROOT_ALIAS>:content/mappers/<entity>.js` |
| `services/<Area>/mappers.js` reexportado pelo barrel | mapper no lugar errado; barrel deixa de ser puro | `content/mappers/`, importado pelo service |
| Mesmo mapper em dois lugares (módulo e pacote compartilhado, ou dois módulos) | cópias divergem | uma fonte só; pacote compartilhado que já exporta tem preferência |
| `delete body.<field>` pra omitir campo | muta objeto; difícil de ler | spread condicional no mapper |
| Payload montado à mão no service/store | regra de transformação espalhada | `map<Entity>Input` |
| Mapper chamando `rest` ou lendo store | deixa de ser puro; teste precisa de mock | mapper recebe tudo por argumento |

## Referências cruzadas

- [[Playbook/services]]
- [[Playbook/visao-profile]]
- [[Playbook/new-module]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/mapper.js]]
