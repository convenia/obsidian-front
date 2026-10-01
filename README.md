---
tags:
  - readme
  - instalacao
  - agentes
date: 2026-10-01
---

# obsidian-frontend

Vault Obsidian que serve como **referencia preferencial de estilo de codigo frontend** das SPAs Convenia (spa-admin, spa-colab, etc.) para devs e agentes de IA (Claude Code, Codex). Ponto de entrada do conteudo: [[home]].

> [!warning] Status: beta
> O padrao documentado aqui ainda esta em construcao. Ele pode divergir do codigo real. Ver [[#Politica de atualizacao (beta)]].

## Estrutura

- [[AGENTS]] — regras de escrita do vault (lidas pelo agente ao editar o vault).
- [[Playbook/playbook]] — guias: o que fazer, por que, red flags.
- [[Templates/templates]] — esqueletos de codigo prontos pra copiar.
- [[Projects/projects]] — projetos/features ativos.

## Instalar como referencia para agentes

Instalar = colar o snippet abaixo num arquivo de instrucoes que o agente carrega automaticamente. Escolha **um** escopo por ferramenta.

Premissa: o vault esta clonado em `~/Desktop/Convenia/obsidian-frontend`. Se estiver em outro lugar, troque os caminhos no snippet.

### Snippet

```markdown
## Convenia Frontend — referencia de estilo (beta)

Escopo: projetos dentro de `~/Desktop/Convenia/` que tenham codigo frontend (spa-admin, spa-colab, etc.).

### Requisitos

- Antes de escrever, refatorar ou revisar codigo frontend nesses projetos, o agente SHOULD ler o vault `~/Desktop/Convenia/obsidian-frontend` como referencia preferencial de estilo.
- Ordem de leitura: `AGENTS.md` → `home.md` → `Playbook/playbook.md` → guia especifico da camada → `Playbook/red-flags.md`.
- Para gerar codigo, o agente SHOULD partir do template correspondente em `Templates/`.
- Em conflito, a prioridade e: instrucao explicita do usuario > CLAUDE.md/AGENTS.md do projeto > padrao do codigo existente no repo > vault. O agente MUST reportar o conflito ao usuario.
- O vault esta em beta. Quando o usuario der uma correcao ou orientacao de estilo/padrao frontend, ou quando o agente detectar divergencia entre vault e codigo real, o agente MUST propor o ajuste no vault (arquivo + resumo da mudanca).
- O agente MUST NOT editar, commitar ou dar push no vault sem autorizacao explicita do usuario.
- Ao editar o vault, o agente MUST seguir `~/Desktop/Convenia/obsidian-frontend/AGENTS.md`.
```

### Claude Code

| Escopo | Arquivo | Vale para |
| --- | --- | --- |
| Global | `~/.claude/CLAUDE.md` | Todas as sessoes. O snippet se restringe a `~/Desktop/Convenia/` pela linha "Escopo". |
| Pasta Convenia | `~/Desktop/Convenia/CLAUDE.md` ou `~/Desktop/Convenia/.claude/CLAUDE.md` | Qualquer projeto dentro de `~/Desktop/Convenia/`. O Claude Code carrega CLAUDE.md dos diretorios ancestrais do diretorio de trabalho. |
| Projeto especifico | `<projeto>/CLAUDE.md` (versionado) ou `<projeto>/CLAUDE.local.md` (pessoal, adicionar ao `.gitignore`) | So aquele projeto. |

Passos:

1. Abrir o arquivo do escopo escolhido. Criar se nao existir.
2. Colar o snippet no fim do arquivo. Se o arquivo ja existir (ex.: `~/Desktop/Convenia/.claude/CLAUDE.md` do AIOX), acrescentar, nao sobrescrever.
3. Abrir uma sessao nova dentro de um projeto (ex.: `cd ~/Desktop/Convenia/spa-admin && claude`).
4. Rodar `/memory` e confirmar que o arquivo aparece na lista de memorias carregadas.

### Codex

| Escopo | Arquivo | Vale para |
| --- | --- | --- |
| Global | `~/.codex/AGENTS.md` | Todas as sessoes. O snippet se restringe a `~/Desktop/Convenia/` pela linha "Escopo". |
| Projeto especifico | `<projeto>/AGENTS.md` | So aquele projeto. |

Passos:

1. Abrir o arquivo do escopo escolhido. Criar se nao existir.
2. Colar o snippet no fim do arquivo, sem apagar o conteudo existente.
3. Abrir uma sessao nova dentro do projeto e perguntar ao agente qual referencia de estilo frontend ele usa.

Restricao: o Codex procura `AGENTS.md` da raiz do repositorio git ate o diretorio atual. Ele nao le `~/Desktop/Convenia/AGENTS.md`, porque essa pasta fica acima da raiz dos repos. Para cobrir todos os projetos Convenia no Codex, use o escopo global.

### Falhas comuns

- Agente nao segue o vault: sessao aberta antes da instalacao. Abrir sessao nova.
- Agente nao acha os arquivos: caminho do vault diferente de `~/Desktop/Convenia/obsidian-frontend`. Corrigir os caminhos no snippet.
- Instrucoes duplicadas: snippet colado em global e em pasta/projeto ao mesmo tempo. Manter um escopo so.

## Politica de atualizacao (beta)

- Correcoes de estilo dadas a qualquer agente durante o trabalho em projetos Convenia SHOULD virar ajuste no vault.
- O agente propoe o ajuste. O usuario autoriza. So depois o agente edita.
- Toda edicao segue [[AGENTS]] e confere se outras notas ficaram contraditorias.

## Referencias cruzadas

- [[home]]
- [[AGENTS]]
- [[Playbook/playbook]]
- [[Playbook/red-flags]]
- [[Templates/templates]]
