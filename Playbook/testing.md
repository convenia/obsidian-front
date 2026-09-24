---
tags:
  - playbook
  - frontend
  - testing
  - playwright
date: 2026-09-24
---

# Testing — Playwright E2E

> Resolver os tokens pela [[Playbook/visao-profile]] antes de aplicar este guia. O shape de teste e identico entre visoes — so os valores mudam.

## O que e

Como o lado SPA de uma migracao de card e validado: specs de integracao Playwright. O organism possui os testes unitarios de componente no submodule.

## Escopo

O lado SPA e validado por specs de integracao Playwright — nao crie teste unitario novo pro container. Regra de field (validacao, mask, `hide` condicional, obrigatoriedade) e territorio de teste unitario do organism, nao Playwright. Renderizar um valor de custom field ponta a ponta (fetch → store → widget do organism) e integracao e entra no set canonico.

## Layout

```
<spec root>/<Area>/<Group>/
├── data/            # fixtures *.json (Config, GetEmployeeFields, Get/Update/Create/Delete<Entity>, +Error, +CustomFields, GetCities)
├── pages/
│   └── <Group>Page.js   # a POM — navegacao + mock de API, SEM seletor
└── specs/
    ├── setup.js         # fixture + tags do grupo + mock padrao no beforeEach
    └── <Card>.spec.js   # um por card; irmaos reusam a mesma POM + setup
```

Cards irmaos de uma area compartilham uma unica POM e um unico `setup.js`. O `<Area>` do spec root precisa ser o mesmo valor usado em `containers/<Area>/`, `services/<Area>/` e `store/<Area>/` — ver [[Playbook/red-flags]] *drift de `<Area>`*.

## Fixture de fields

A fixture de fields e JSON estatico — nunca uma factory programatica (`mocks/GetEmployeeFields.js`) pro payload de fields, isso desvia do contrato. A fixture carrega so `data.sections`; o formato legado `data.fields` cai fora. Como o fetch unificado retorna todas as sections, a fixture compartilhada carrega todo sub-card da area — nao existe fixture de fields por card.

## Set canonico single-item (seis testes, nesta ordem)

1. **`@always`** — renderiza o card e abre o modal de edicao com os system fields.
2. Renderiza todo valor de custom e system field no card e no modal — um teste consolidado (nao dividir). Precisa cobrir **um custom field por tipo** que o organism suporta: `string` (texto), `list` (single-select), `multiple` (multi-select), `boolean` (radio), `date`. Reusar os mocks de Storybook do organism como fonte — nunca inventar shape/valor de campo.
3. Descarta mudancas ao cancelar o modal de edicao.
4. Atualiza system fields com sucesso — validacao escondida apos submit valido, feedback de sucesso, card reflete o novo valor.
5. Atualiza custom fields com sucesso — submit, feedback, reabrir, valida persistencia.
6. Mostra feedback de erro e mantem o estado do modal quando o update falha.

Card multi-item segue o mesmo set **mais** um caso de visibilidade com 2-3 itens.

## Seletores e asserts

Ficam inline na spec + helpers compartilhados, nunca na POM. Inline todo testid como string literal no locator — nunca hoiste um id de field pra `const`/objeto e interpole (`getByTestId(\`select-${cf.id}\`)`).

| Proposito | Locator |
|---|---|
| Container do card | `<CARD_CLASS>` |
| Modal de edicao | `.c-form-builder-modal` |
| Abrir editor | `getByTestId('info-edit-button').first()` |
| Select | `getByTestId('select-<uuid>')` + `toContainText` |
| Radio (boolean) | `getByTestId('option-<uuid>-Sim'\|'-Nao')` + `toHaveClass(/checked/)` |
| Submit/Cancel | `getByTestId('form-builder-modal-submit-button'\|'-cancel-button')` |

## Cards com attachment — upload e remove sao obrigatorios

Trigger e "o card tem attachment", pra single-item e multi-item. Quando tem, adicione dois testes alem do set canonico: substituir attachment (POST + DELETE) e remover attachment (DELETE apenas). Ambos disparam a confirmacao de remocao e verificam por contagem de request via `createDocumentFilesRequestTracker` + `expectRequests`.

## Fixtures de update — sequencia no mesmo path, corpo real de POST

`mockFromList` chaveia uma rota por `metodo + hash(postData)`, e rotas Playwright sao LIFO — uma fixture de update por teste **sombreia** o GET compartilhado da entidade. A fixture precisa ser uma sequencia no mesmo path incluindo o GET inicial: `[GET (entidade inicial), POST (corpo exato), GET (entidade atualizada)]`. O corpo do POST precisa ser **exatamente igual** ao output do write mapper do organism — nunca adivinhe o corpo a mao; capture com um logger temporario em `page.on('request', ...)`, rode uma vez, fixe o valor, remova o logger.

## Dividindo uma spec de card inchada

Uma spec que acumula o set canonico + testes de attachment + cobertura completa de tipo de custom field pode passar de 10 testes num unico arquivo `mode: 'serial'`, virando o gargalo de teste lento. Divida por dependencia de mock compartilhado (nao por contagem arbitraria), mantendo o mesmo `setup.js`/POM em todo arquivo novo, cada um com exatamente um teste `@always`. Antes de tirar `mode: 'serial'` de um arquivo dividido, rode com `--repeat-each=5 --workers=4` no arquivo isolado e depois `--repeat-each=3 --workers=8` no grupo inteiro — os dois precisam passar em toda repeticao.

## Referencias cruzadas

- [[Playbook/visao-profile]]
- [[Playbook/container]]
- [[Playbook/red-flags]]
- [[Templates/Codigo/spec.playwright]]
