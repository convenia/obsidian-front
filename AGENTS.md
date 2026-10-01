---
tags:
  - agentes
  - regras
  - vault
date: 2026-09-24
---

# AGENTS

## Regras obrigatorias

- Sempre responder e escrever em portugues brasileiro.
- Sempre usar links internos em formato wikilink, como [[home]].
- Toda nota Markdown precisa ter frontmatter com pelo menos `tags` e `date`.
- Toda nota Markdown segue o formato limpo: frontmatter sem recuo, titulo na coluna 1, secoes em `##` na coluna 1, listas com um nivel de indentacao.
- Nao deixar frontmatter, headings, bullets ou paragrafos com recuo artificial de quatro espacos.
- Nomes de arquivos sem acento e sem espaco, usando hifens quando necessario.
- Termos tecnicos (nomes de arquivo, tokens, trechos de codigo) ficam em ingles mesmo dentro de prosa em portugues.
- Cada guia de [[Playbook/playbook]] termina em secao "Referencias cruzadas", linkando o template de codigo correspondente e os outros guias relacionados.
- Escrever as anotacoes num nivel aproximado pra quem e junior ou nao conhece o front, quando necessario — explicar termo tecnico e o "porque" antes de assumir que o leitor ja sabe.

## Contexto do vault

- Este vault documenta o padrao **generico** de arquitetura frontend usado pelas SPAs Convenia (spa-admin, spa-colab) quando um card de tela consome um organism de `@convenia/components` — vale pra qualquer card/dominio, nao um card especifico.
- A fonte original desse padrao sao as skills `spas-reference` e `system-custom-fields-organism`, mantidas no submodule `.@convenia` do repo `spa-colab` (`src/organisms/Employee/SystemFields/specs/skills/`). Este vault nao copia essas skills: destrincha o padrao delas em guias e templates proprios generalizados. Onde o caso real (System+Custom Fields) ajuda a ilustrar, ele aparece rotulado "Exemplo real" — nunca como a regra padrao.
- A pasta [[Playbook/playbook]] concentra os guias de orientacao (o que, por que, red flags).
- A pasta [[Templates/templates]] contem os esqueletos de codigo prontos pra copiar.
- A pasta [[Projects/projects]] organiza projetos/features frontend ativos.

## Como trabalhar

- Antes de migrar ou construir um card, ler [[Playbook/visao-profile]] primeiro — ele define os tokens parametrizaveis (`<ROOT>`, `<TYPE_PREFIX>`, `<STORE_MODULE>`, etc.) que os outros guias usam.
- Para gerar codigo (container, service, store, teste), comecar pelo template correspondente em [[Templates/templates]] e resolver os tokens usando o profile.
- Antes de fechar qualquer tarefa, conferir contra [[Playbook/red-flags]] — cada item ali e uma violacao conhecida do padrao.
- Registrar contexto, objetivo e decisoes na nota correta de [[Projects/projects]].
- Sempre que uma nota for atualizada, verificar se outras notas do vault ficam contraditorias e se alguma parte do codigo real (spa-admin, spa-colab) precisa de ajuste correspondente.
