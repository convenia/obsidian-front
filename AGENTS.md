---
tags:
  - agentes
  - regras
  - vault
date: 2026-10-01
---

# AGENTS

## Regras obrigatórias

- Sempre responder e escrever em português brasileiro.
- Prosa MUST usar acentuação correta do pt-BR (é, não, só, função, módulo). Isso vale para parágrafos, listas, tabelas, headings e comentários dentro de blocos de código.
- Sempre usar links internos em formato wikilink, como [[home]].
- Toda nota Markdown precisa ter frontmatter com pelo menos `tags` e `date`.
- Toda nota Markdown segue o formato limpo: frontmatter sem recuo, título na coluna 1, seções em `##` na coluna 1, listas com um nível de indentação.
- Não deixar frontmatter, headings, bullets ou parágrafos com recuo artificial de quatro espaços.
- Nomes de arquivos sem acento e sem espaço, usando hífens quando necessário. Wikilinks, tags do frontmatter, tokens (`<ROOT>`) e identificadores de código também ficam sem acento.
- Termos técnicos (nomes de arquivo, tokens, trechos de código) ficam em inglês mesmo dentro de prosa em português.
- Cada guia de [[Playbook/playbook]] termina em seção "Referências cruzadas", linkando o template de código correspondente e os outros guias relacionados.
- Escrever as anotações num nível aproximado pra quem é júnior ou não conhece o front, quando necessário — explicar termo técnico e o "porquê" antes de assumir que o leitor já sabe.

## Contexto do vault

- Este vault documenta o padrão **genérico** de arquitetura frontend usado pelas SPAs Convenia (spa-admin, spa-colab) quando um card de tela consome um organism de `@convenia/components` — vale pra qualquer card/domínio, não um card específico.
- A fonte original desse padrão são as skills `spas-reference` e `system-custom-fields-organism`, mantidas no submodule `.@convenia` do repo `spa-colab` (`src/organisms/Employee/SystemFields/specs/skills/`). Este vault não copia essas skills: destrincha o padrão delas em guias e templates próprios generalizados. Onde o caso real (System+Custom Fields) ajuda a ilustrar, ele aparece rotulado "Exemplo real" — nunca como a regra padrão.
- A pasta [[Playbook/playbook]] concentra os guias de orientação (o que, por que, red flags).
- A pasta [[Templates/templates]] contém os boilerplates de código prontos pra copiar.
- A pasta [[Projects/projects]] organiza projetos/features frontend ativos.

## Como trabalhar

- Antes de migrar ou construir um card, ler [[Playbook/visao-profile]] primeiro — ele define os tokens parametrizáveis (`<ROOT>`, `<TYPE_PREFIX>`, `<STORE_MODULE>`, etc.) que os outros guias usam.
- Para gerar código (container, service, store, teste), começar pelo template correspondente em [[Templates/templates]] e resolver os tokens usando o profile.
- Antes de fechar qualquer tarefa, conferir contra [[Playbook/red-flags]] — cada item ali é uma violação conhecida do padrão.
- Registrar contexto, objetivo e decisões na nota correta de [[Projects/projects]].
- Sempre que uma nota for atualizada, verificar se outras notas do vault ficam contraditórias e se alguma parte do código real (spa-admin, spa-colab) precisa de ajuste correspondente.
