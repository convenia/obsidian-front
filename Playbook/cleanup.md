---
tags:
  - playbook
  - frontend
  - cleanup
date: 2026-09-24
---

# Cleanup — remover artefato morto

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia.

## O que e

O sweep deterministico que roda como **ultima etapa**, depois que service/store/loader/container/teste estao verdes, pra remover o codigo morto que a migracao pra V2 deixa pra tras.

## Regra dura — grep antes de apagar

Antes de remover qualquer artefato nomeado (type, action, query, const, service, mock, fixture), de grep no nome no repo inteiro. Apague so se esta migracao foi o **ultimo** consumidor. Se ainda e despachado/importado em outro lugar (comumente um modulo comum compartilhado, ou em admin, Admission), mantenha a definicao e remova so as referencias **desta** area.

## O que remover

- **Types de store** — os types legados de write/file por entidade que as actions unificadas substituem, os types de eixo separado (`GET/SET_{SYSTEM,CUSTOM}_FIELDS`), e qualquer `<TYPE_PREFIX>_<AREA>_CUSTOM_FIELDS`/`GET_REQUIRED_SYSTEM_FIELDS`. Reordene o bloco tocado (ver [[Playbook/store]]).
- **Services** — fetches legados substituidos: `getEmployeeInfoSchema`, `getRequiredSystemFields`, `updateAdditionalInfo`, qualquer `services/<Area>/mappers.js`/`forms.js` da SPA, qualquer uso de `resolveCustomFieldValues`.
- **Form schemas da SPA** — o organism agora possui a construcao do form, entao o modulo de form da SPA do card migrado esta morto. Apague e remova a entrada do index agregador. De grep antes — um card irmao ainda legado mantem o proprio form ate migrar.
- **Queries/options GraphQL** — uma query ou fetch de options que nao e mais disparado da area migrada. Remova a definicao, o type/action de store, o state de `options` **e** o mock de teste, juntos.
- **Consts da SPA** — qualquer const de label/section-id/uuid que agora mora no organism. Apague a copia da SPA e importe do organism — mantenha so ids genuinamente locais que o organism nao fornece.
- **Testes** — metodos de POM, fixtures `data/*.json` e wrappers de mock de rotas/queries que voce acabou de remover.

## Nao marque `@deprecated`

Remova o codigo morto de fato. Nao deixe marcador `@deprecated`, bloco comentado, ou arquivo "mantido pra referencia" — isso le como ainda suportado.

## Pos-condicao

- O app compila cross-feature (todo nome removido nao tem dispatcher/import restante).
- As specs do card passam sem mock apontando pra rota/query deletada.
- Nao sobra copia da SPA de um const/mapper/label que o organism ja fornece.

## Referencias cruzadas

- [[Playbook/red-flags]]
- [[Playbook/store]]
- [[Playbook/services-and-mappers]]
