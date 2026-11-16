---
tags:
  - playbook
  - frontend
  - fontes
  - indice
date: 2026-11-16
---

# Fontes — vault x playbook da Convenia

## O que e

O mapa entre os guias deste vault e o playbook da Convenia (`convenia/playbook`). Responde tres perguntas: onde o playbook trata o mesmo assunto, o que so existe aqui, e onde os dois divergem hoje.

Os caminhos sao relativos a `convencoes-de-desenvolvimento/front-end/` no repo do playbook, no formato `arquivo.md:linha-inicial-linha-final`. Intervalos conferidos no commit `bb813f2` do playbook (2026-11-12) — linha muda com o tempo; se o trecho nao bater, procurar pelo titulo da secao.

## Por que existe

O playbook e a autoridade; o vault aprofunda um padrao. Sem este mapa, uma regra do vault parece ter o mesmo peso de uma convencao do time inteiro, e uma divergencia entre os dois so aparece quando alguem tropeca nela num review.

## Guia do vault → trecho do playbook

| Guia do vault | Assunto no playbook | Trecho |
|---|---|---|
| [[Playbook/container]] | Container x componente de apresentacao | `introducao/container-component.md:7-25` |
| [[Playbook/container]] | Containers na arquitetura | `introducao/referencia-arquitetura.md:319-332` |
| [[Playbook/container]] | Composition API em container, store e router por stack | `introducao/composition-api.md:7-124` |
| [[Playbook/services-and-mappers]] | Services | `introducao/referencia-arquitetura.md:411-588` |
| [[Playbook/store]] | Store | `introducao/referencia-arquitetura.md:589-853` |
| [[Playbook/loader]] | Composables e `useLoaders` | `introducao/composition-api.md:314-369` |
| [[Playbook/component-conventions]] | Regras de desenvolvimento da lib | `lib-compartilhada/requisitos-tecnicos.md:11-37` |
| [[Playbook/component-conventions]] | Storybook | `lib-compartilhada/storybook.md:12-51` |
| [[Playbook/component-conventions]] | Comentarios e padroes de JavaScript | `introducao/padroes-javascript.md:1-97` |
| [[Playbook/testing]] | Qual teste usar | `testes/escolha.md:10-19` |
| [[Playbook/testing]] | POM (`mockAPIData`, `start`) | `testes/integracao.md:65-117` |
| [[Playbook/testing]] | Teste de falha de requisicao | `testes/integracao.md:296-391` |
| [[Playbook/testing]] | Tag `@always` | `testes/integracao.md:398-400` |
| [[Playbook/testing]] | Cobertura minima | `testes/cobertura.md:53-73` |
| [[Playbook/new-module]] | Estrutura de pastas e import aliases | `introducao/referencia-arquitetura.md:25-78` |
| [[Playbook/new-module]] | Rotas | `introducao/referencia-arquitetura.md:108-191` |
| [[Playbook/review]] | Revisao de codigo, commits, branches, labels | `processos-produto/revisao-de-codigo.md:1-109` |
| [[Playbook/review]] | Falhas graves | `processos-produto/falhas-graves.md:1-17` |

## So existe no vault

Regras do padrao card+organism que o playbook nao trata. Valem dentro desse padrao; fora dele sao sugestao (ver [[Playbook/review]]).

- Tokens parametrizaveis por visao — [[Playbook/visao-profile]].
- Fetch unificado de metadado, mapper de leitura e de escrita no organism — [[Playbook/services-and-mappers]].
- Convencao de types `<TYPE_PREFIX>_<VERB>_[<AREA>_]<ENTITY>` e ordem alfabetica de state/mutations/actions — [[Playbook/store]].
- Contrato de evento `@submit`/`@remove` com `callback` e action unica de update — [[Playbook/container]].
- Ordem alfabetica de props, emits e atributos de template; `Object.freeze` em consts de enum — [[Playbook/component-conventions]].
- Set canonico de seis testes por card e fixture em sequencia no mesmo path — [[Playbook/testing]].
- Sweep de codigo legado — [[Playbook/cleanup]].

## Divergencias conhecidas

Onde o vault e o playbook dizem coisas diferentes. Ate a nota do vault ser corrigida, vale o playbook.

| Assunto | Vault | Playbook | Situacao |
|---|---|---|---|
| Import de `@convenia/components` | [[Playbook/component-conventions]] pedia sempre o barrel raiz, nunca path profundo | Caminho profundo em vez do barrel — `lib-compartilhada/requisitos-tecnicos.md:14-21` | Nota ajustada: a regra do barrel vale pros pacotes de organisms; pra `@convenia/components` vale o playbook |
| Acesso a store e router | [[Playbook/container]] fixava o macro `vuex-composition.macro` e `vue-composition-wrapper` pra todo container | Depende da stack do SPA: macro so em Vue 2 + Vuex 3; helpers nativos em Vue 3 + Vuex 4; Pinia com `defineStore` — `introducao/composition-api.md:39-124` | Nota ajustada pra citar a stack |
| Comentarios | [[Playbook/component-conventions]] proibe comentario, com duas excecoes | "Comente o porque, nao o que" — `introducao/padroes-javascript.md:9` | Vault e mais restrito. Em aberto: o time confirma se a regra restrita vale so pra lib ou pra SPA tambem |
| Teste unitario de container | [[Playbook/testing]] dizia pra nunca criar teste unitario de container | Unitario em SPA e reservado pra logica pura que a integracao nao alcanca; wiring coberto pelo Playwright nao ganha unitario — `testes/escolha.md:15-17` | Nota ajustada: container fino de card nao tem logica propria, por isso nao ganha unitario |

## Como manter

- Guia novo no vault entra na primeira tabela, ou na lista "So existe no vault".
- Achou divergencia num review: registrar na terceira tabela no mesmo PR que corrige a nota.
- Ao conferir os intervalos de novo, atualizar o commit e a data no topo desta nota.

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/review]]
- [[Playbook/red-flags]]
