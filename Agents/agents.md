---
tags:
  - agentes
  - indice
date: 2026-09-24
---

# Agents

Indice de agentes/triagem deste vault. Vazio por enquanto — pasta criada seguindo o esqueleto do vault backend, pronta pra crescer quando houver agentes de triagem especificos do frontend (ex.: triagem de falha de teste Playwright, triagem de erro Sentry do frontend).

## Agente de review de PR frontend

Um agente que revisa PR de SPA depois que os testes passam e comenta os achados no proprio PR. Como ele usa este vault:

- Entra por [[Playbook/review]], que define a ordem das fontes: playbook da Convenia primeiro, este vault como complemento.
- Usa [[Playbook/red-flags]] como checklist, com a coluna "Como detectar" pra achar o ponto no diff e a coluna "Fonte" pra citar.
- Usa [[Playbook/fontes]] pra localizar o trecho do playbook de cada assunto e pra saber onde o vault diverge dele.
- Regra marcada `so vault` nunca bloqueia PR fora do padrao card+organism.

## Referencias cruzadas

- [[home]]
- [[Playbook/review]]
