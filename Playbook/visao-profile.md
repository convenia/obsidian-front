---
tags:
  - playbook
  - frontend
  - tokens
date: 2026-09-24
---

# Visao Profile

## O que e

O modelo de parametros que permite um unico padrao de SPA (container + store + service + loader + teste) servir varias "visoes" — SPAs ou papeis diferentes que consomem o mesmo organism compartilhado. Todo guia deste Playbook e escrito contra esses tokens — antes de aplicar qualquer guia, resolva os tokens pra sua visao.

A tecnica (parametrizar por visao) e generica — qualquer organism consumido por mais de uma SPA/papel se beneficia dela. A tabela abaixo usa **spa-admin / spa-colab-self** como exemplo concreto real (as duas visoes que hoje consomem cards System+Custom Fields); pra um organism novo, preencha as colunas com as visoes que ele de fato tem.

## Por que existe

A arquitetura e UNIFICADA: existe um unico formato de container, store e teste pra qualquer visao — nao ha eixo de variante estrutural. O que muda entre visoes sao apenas os valores abaixo. Isso evita reinventar o padrao a cada nova SPA ou area.

Se um token nao tiver binding definido pra sua visao, pare — o padrao nao esta totalmente descrito pra essa superficie ainda; nao adivinhe um valor.

## Tokens

| Token | Significado | spa-admin | spa-colab-self |
|---|---|---|---|
| `<ROOT>` | pasta raiz da feature | `Employee` | `Information` |
| `<ROOT_ALIAS>` | prefixo do import alias | `@Employee:` | `@Information:` |
| `<TYPE_PREFIX>` | prefixo do nome dos types da store | `EMPLOYEE` | `INFO` |
| `<TYPE_NS>` | namespace dos types | `employee/` | `information/` |
| `<STORE_MODULE>` | nome do modulo Vuex | `employee<Area>` | `information<Area>` |
| `<MODULE_REG>` | como o modulo e registrado | `meta.storeModules` da rota | export no barrel `src/Information/store/index.js` |
| `<VISAO>` | segmento de role no `<METADATA_ENDPOINT>` | fixo `admin` | param `route`, default `'employee'` |
| `<REST_MW>` | import do middleware REST | `@modules/request/middlewareRest` | `@modules/http/middlewareRest` |
| `<ID_SOURCE>` | origem do id da empresa/colaborador | `companyUuid` de `@modules/authHelpers` | `COMPANY_UUID`/`EMPLOYEE_UUID` de `@src/cookies` |
| `<CAN_EDIT_EXPR>` | como deriva `canEdit` | `!isDisconnected` (modulo `employee`) | fixo `true` no card V2 migrado |
| `<CONTAINER_DIR>` | pasta do container do card | `containers/Details/<Area>/` | `containers/<Area>/` |
| `<ENTITY_BASE>` | URL base de GET/write da entidade | `.../employees/{employeeId}/<resource>/admin` | `.../employees/{employeeId}/<resource>/{route}` |
| `<CITIES_EP>` | endpoint de cidades dependentes | `/states/{stateId}/cities` | `/states/{stateId}/cities/options` |
| `<NAV_URL>` | URL de navegacao do teste | `/colaboradores/<employeeId>/detalhes/<area>` | `/minhas-informacoes` |
| `<TEST_SETUP>` | base test + fixture Playwright | `@tests:setups/logged` (`logged`) | `@tests:setups/base` (`base`) |
| `<GROUP_TAG>` | tags do describe | `[ '@Employee', '@EmployeeDetails' ]` | `[ '@Information' ]` |
| `<CARD_CLASS>` | seletor CSS do container do card | `.employee-<area>-container` | `.employee-information-<area>-...-container` |
| `<HELPERS_PATH>` | path dos helpers de teste compartilhados | `@tests/specs/Employees/shared/helpers/*` | `@tests/specs/Information/shared/helpers/*` |
| `<CARD_GATE>` | gate condicional do card inteiro | predicate do organism (`helpers`) | computed do parent sobre estado ja carregado |
| `<HAS_ADMISSION>` | existe trilha de Admission | sim | nao |
| `<BRANCH_GATE>` | prerequisito de branch do submodule | confirmacao generica | gate explicito de prerequisito |

`<VISAO>` e um binding por **visao**, nao por area — a familia colab usa um `route` param (self default `'employee'`; gestor/Area Manager usa `'supervisor'`).

## Como consumir

1. Publicar uma tabela de Visao Profile no ponto de entrada da tarefa, ligando cada token acima ao valor da visao em questao.
2. Ler cada guia deste Playbook substituindo os tokens pelos bindings resolvidos.

## Referencias cruzadas

- [[Playbook/playbook]]
- [[Playbook/services-and-mappers]]
- [[Playbook/store]]
- [[Playbook/container]]
- [[Playbook/testing]]
