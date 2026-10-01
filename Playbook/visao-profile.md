---
tags:
  - playbook
  - frontend
  - tokens
date: 2026-09-24
---

# Visão Profile

## O que é

O modelo de parâmetros que permite um único padrão de SPA (container + store + service + loader + teste) servir várias "visões" — SPAs ou papéis diferentes que consomem o mesmo organism compartilhado. Todo guia deste Playbook é escrito contra esses tokens — antes de aplicar qualquer guia, resolva os tokens pra sua visão.

A técnica (parametrizar por visão) é genérica — qualquer organism consumido por mais de uma SPA/papel se beneficia dela. A tabela abaixo usa **spa-admin / spa-colab-self** como exemplo concreto real (as duas visões que hoje consomem cards System+Custom Fields); pra um organism novo, preencha as colunas com as visões que ele de fato tem.

## Por que existe

A arquitetura é UNIFICADA: existe um único formato de container, store e teste pra qualquer visão — não há eixo de variante estrutural. O que muda entre visões são apenas os valores abaixo. Isso evita reinventar o padrão a cada nova SPA ou área.

Se um token não tiver binding definido pra sua visão, pare — o padrão não está totalmente descrito pra essa superfície ainda; não adivinhe um valor.

## Tokens

| Token | Significado | spa-admin | spa-colab-self |
|---|---|---|---|
| `<ROOT>` | pasta raiz da feature | `Employee` | `Information` |
| `<ROOT_ALIAS>` | nome do import alias do módulo (uso: `<ROOT_ALIAS>:content/...`) | `@Employee` | `@Information` |
| `<TYPE_PREFIX>` | prefixo do nome dos types da store | `EMPLOYEE` | `INFO` |
| `<TYPE_NS>` | namespace dos types | `employee/` | `information/` |
| `<STORE_MODULE>` | nome do módulo Vuex | `employee<Area>` | `information<Area>` |
| `<MODULE_REG>` | como o módulo é registrado | `meta.storeModules` da rota | export no barrel `src/Information/store/index.js` |
| `<VISAO>` | segmento de role no `<METADATA_ENDPOINT>` | fixo `admin` | param `route`, default `'employee'` |
| `<REST_PATH>` | import do middleware REST | `@modules/request/middlewareRest` | `@modules/http/middlewareRest` |
| `<ID_SOURCE>` | origem do id da empresa/colaborador | `companyUuid` de `@modules/authHelpers` | `COMPANY_UUID`/`EMPLOYEE_UUID` de `@src/cookies` |
| `<CAN_EDIT_EXPR>` | como deriva `canEdit` | `!isDisconnected` (módulo `employee`) | fixo `true` no card V2 migrado |
| `<CONTAINER_DIR>` | pasta do container do card | `containers/Details/<Area>/` | `containers/<Area>/` |
| `<ENTITY_BASE>` | URL base de GET/write da entidade | `.../employees/{employeeId}/<resource>/admin` | `.../employees/{employeeId}/<resource>/{route}` |
| `<CITIES_EP>` | endpoint de cidades dependentes | `/states/{stateId}/cities` | `/states/{stateId}/cities/options` |
| `<NAV_URL>` | URL de navegação do teste | `/colaboradores/<employeeId>/detalhes/<area>` | `/minhas-informacoes` |
| `<TEST_SETUP>` | base test + fixture Playwright | `@tests:setups/logged` (`logged`) | `@tests:setups/base` (`base`) |
| `<GROUP_TAG>` | tags do describe | `[ '@Employee', '@EmployeeDetails' ]` | `[ '@Information' ]` |
| `<CARD_CLASS>` | seletor CSS do container do card | `.employee-<area>-container` | `.employee-information-<area>-...-container` |
| `<HELPERS_PATH>` | path dos helpers de teste compartilhados | `@tests/specs/Employees/shared/helpers/*` | `@tests/specs/Information/shared/helpers/*` |
| `<CARD_GATE>` | gate condicional do card inteiro | predicate do organism (`helpers`) | computed do parent sobre estado já carregado |
| `<HAS_ADMISSION>` | existe trilha de Admission | sim | não |
| `<BRANCH_GATE>` | pré-requisito de branch do submodule | confirmação genérica | gate explícito de pré-requisito |

`<VISAO>` é um binding por **visão**, não por área — a família colab usa um `route` param (self default `'employee'`; gestor/Area Manager usa `'supervisor'`).

## Como consumir

1. Publicar uma tabela de Visão Profile no ponto de entrada da tarefa, ligando cada token acima ao valor da visão em questão.
2. Ler cada guia deste Playbook substituindo os tokens pelos bindings resolvidos.

## Referências cruzadas

- [[Playbook/playbook]]
- [[Playbook/services]]
- [[Playbook/mappers]]
- [[Playbook/store]]
- [[Playbook/container]]
- [[Playbook/testing]]
