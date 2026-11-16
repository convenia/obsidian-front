---
tags:
  - playbook
  - frontend
  - review
  - checklist
date: 2026-11-16
---

# Review de PR frontend

## O que e

Como revisar um PR de SPA usando este vault junto com o playbook da Convenia (`convenia/playbook`). Serve pra revisor humano e pra agente de review. Os outros guias dizem como construir; este diz como conferir o que foi construido e com que peso cobrar cada regra.

## Ordem das fontes

1. **Playbook da Convenia** — e a autoridade. Toda convencao citada num review aponta pro trecho do playbook. O mapa de onde cada assunto mora esta em [[Playbook/fontes]].
2. **Instrucoes do proprio repo** — `AGENTS.md`, `CLAUDE.md`, `.@convenia/AGENTS-spa.md`.
3. **Este vault** — fonte complementar. Aprofunda um padrao que o playbook nao detalha: o card de SPA que consome um organism de `@convenia/components`.

Regras pra misturar as fontes:

- Quando o vault e o playbook divergem, vale o playbook. Registre a divergencia no review e abra a correcao aqui no vault — as divergencias conhecidas ficam listadas em [[Playbook/fontes]].
- Regra marcada `so vault` em [[Playbook/red-flags]] vale como bloqueio so dentro do padrao card+organism. Fora dele, e sugestao.
- Nunca afirme "a convencao e X" sem trecho que sustente. Sem fonte, diga que e opiniao do revisor.

## Antes de ler o diff

- Ler o card/issue e a descricao do PR. PR que nao faz o que foi pedido esta errado, por mais limpo que o codigo esteja.
- Identificar o regime do repo no `package.json`: versao do Vue, Vuex ou Pinia, `<script setup>` ou Options API nos arquivos vizinhos. A mesma linha e certa num SPA e errada em outro — a forma de acessar store e router muda por stack (`introducao/composition-api.md:39-124`).
- Conferir o tamanho: o playbook recomenda ate 500 linhas adicionadas por PR e sugere separar containers + store + services dos componentes de apresentacao (`processos-produto/revisao-de-codigo.md:57-59`).
- Conferir branch, mensagens de commit e labels (`processos-produto/revisao-de-codigo.md:5-51` e `:94-109`). A label decide quais testes de integracao rodam: sem `test-all` ou `feat/<tag>`, so rodam os `@always`.

## O que conferir no diff

Na ordem em que costuma render mais:

1. **Contrato alterado e consumidor esquecido.** Pra cada prop, emit, funcao exportada, action/getter/type de store ou `data-testid` que mudou, dar grep no identificador (kebab e Pascal pra componente) e abrir os pontos de uso. Mudanca em `.@convenia` alcanca outros SPAs.
2. **Caminho de falha.** Service retorna `[ err, data ]`: o erro e lido? O usuario ve alguma coisa? Request de listagem que falha nunca pode virar empty state (`processos-produto/falhas-graves.md:15-17`).
3. **Separacao container/componente** (`introducao/container-component.md:7-25`). No padrao de card, conferir contra [[Playbook/container]] secao "O que o container NAO deve fazer".
4. **Red flags do padrao de card** — passar a lista de [[Playbook/red-flags]] usando a coluna "Como detectar".
5. **Testes.** A escolha entre unitario e integracao segue o playbook (`testes/escolha.md:10-19`). O teste exercita o que o nome diz? Dependencia stubada nao prova o comportamento que ela substitui. Cada grupo de integracao tem um teste `@always` (`testes/integracao.md:398-400`)? Existe teste do cenario de falha de request (`testes/integracao.md:296-391`)? Ver [[Playbook/testing]].
6. **Estilo e CSS.** Codigo de feature em SPA nao usa `scoped` (`introducao/css.md:27-31`); componente da lib usa (`lib-compartilhada/requisitos-tecnicos.md:23-36`). Classe raiz com o nome do componente (`introducao/css.md:55-71`).
7. **Desenho.** O conserto e proporcional a causa? Criou helper onde ja existe padrao na lib ou em outro SPA? Se a abordagem esta errada, dizer isso primeiro em vez de empilhar ajuste de linha em cima dela.

## Como escrever o achado

- Um achado por comentario, na linha do problema, com o trecho de codigo que o sustenta.
- Comecar pela severidade — as mesmas de [[Playbook/red-flags]]:
  - `bloqueia` — bug, contrato quebrado, caminho de falha ausente, ou desvio de convencao com fonte no playbook.
  - `corrigir` — problema real que sozinho nao justifica outra rodada.
  - `sugestao` — opcional, incluindo tudo que so este vault sustenta fora do padrao de card.
- Dizer o conserto concreto, nao so o defeito.
- Cobrar so o que o PR introduziu. Problema que ja esta na branch base vira debito anotado, nao bloqueio.
- Na segunda rodada, revisar so os commits novos e dizer, pra cada bloqueio anterior, se foi resolvido — lendo o codigo, nao a mensagem do commit. Nao levantar exigencia nova sobre codigo ja aprovado.

## O que este vault nao cobre

Pra estes assuntos, usar direto o playbook (trechos em [[Playbook/fontes]]): CSS e cores, formularios e modal fora do padrao de card, migracao Vue 3, migracao Vuex → Pinia, cobertura minima, mocks com Mirage, Storybook e regressao visual, processo de deploy.

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/fontes]]
- [[Playbook/red-flags]]
- [[Playbook/container]]
- [[Playbook/testing]]
- [[Agents/agents]]
