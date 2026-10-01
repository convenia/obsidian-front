---
tags:
  - readme
  - instalacao
  - agentes
date: 2026-10-01
---

# obsidian-frontend

Vault Obsidian que serve como **referência preferencial de estilo de código frontend para o time de Guará** das SPAs Convenia (spa-admin, spa-colab, etc.) para devs e agentes de IA (Claude Code, Codex). Ponto de entrada do conteúdo: [[home]].

> [!warning] Status: beta
> O padrão documentado aqui ainda está em construção. Ele pode divergir do código real. Ver [[#Política de atualização (beta)]].

## Estrutura

- [[AGENTS]] — regras de escrita do vault (lidas pelo agente ao editar o vault).
- [[Playbook/playbook]] — guias: o que fazer, por que, red flags.
- [[Templates/templates]] — boilerplates de código prontos pra copiar.
- [[Projects/projects]] — projetos/features ativos.

## Instalar como referência para agentes

Instalar = colar o snippet abaixo num arquivo de instruções que o agente carrega automaticamente. Escolha **um** escopo por ferramenta.

Premissa: o vault está clonado em `~/Desktop/Convenia/obsidian-frontend`. Se estiver em outro lugar, troque os caminhos no snippet.

### Snippet

```markdown
## Convenia Frontend — referência de estilo (beta)

Escopo: projetos dentro de `~/Desktop/Convenia/` que tenham código frontend (spa-admin, spa-colab, etc.).

### Requisitos

- Antes de escrever, refatorar ou revisar código frontend nesses projetos, o agente SHOULD ler o vault `~/Desktop/Convenia/obsidian-frontend` como referência preferencial de estilo.
- Ordem de leitura: `AGENTS.md` → `home.md` → `Playbook/playbook.md` → guia específico da camada → `Playbook/red-flags.md`.
- Para gerar código, o agente SHOULD partir do template correspondente em `Templates/`, exceto quando o projeto atual já tiver pattern/template próprio para aquele tipo de código.
- Pattern/template próprio do projeto atual = docs, templates, skills, generators ou implementação existente equivalente (ex.: outro service, store ou container do mesmo repo). Quando existir, o agente MUST seguir o pattern do projeto atual e MUST ignorar o pattern/template equivalente do vault.
- O agente SHOULD usar o vault só para o que o projeto atual não define.
- Em conflito, a prioridade é: instrução explícita do usuário > CLAUDE.md/AGENTS.md do projeto > padrão do código existente no repo > vault. O agente MUST reportar o conflito ao usuário.
- O vault está em beta. Quando o usuário der uma correção ou orientação de estilo/padrão frontend, ou quando o agente detectar divergência entre vault e código real, o agente MUST propor o ajuste no vault (arquivo + resumo da mudança).
- O agente MUST NOT editar, commitar ou dar push no vault sem autorização explícita do usuário.
- Ao editar o vault, o agente MUST seguir `~/Desktop/Convenia/obsidian-frontend/AGENTS.md`.
```

### Claude Code

| Escopo | Arquivo | Vale para |
| --- | --- | --- |
| Global | `~/.claude/CLAUDE.md` | Todas as sessões. O snippet se restringe a `~/Desktop/Convenia/` pela linha "Escopo". |
| Pasta Convenia | `~/Desktop/Convenia/CLAUDE.md` ou `~/Desktop/Convenia/.claude/CLAUDE.md` | Qualquer projeto dentro de `~/Desktop/Convenia/`. O Claude Code carrega CLAUDE.md dos diretórios ancestrais do diretório de trabalho. |
| Projeto específico | `<projeto>/CLAUDE.md` (versionado) ou `<projeto>/CLAUDE.local.md` (pessoal, adicionar ao `.gitignore`) | Só aquele projeto. |

Passos:

1. Abrir o arquivo do escopo escolhido. Criar se não existir.
2. Colar o snippet no fim do arquivo. Se o arquivo já existir (ex.: `~/Desktop/Convenia/.claude/CLAUDE.md` do AIOX), acrescentar, não sobrescrever.
3. Abrir uma sessão nova dentro de um projeto (ex.: `cd ~/Desktop/Convenia/spa-admin && claude`).
4. Rodar `/memory` e confirmar que o arquivo aparece na lista de memórias carregadas.

### Codex

| Escopo | Arquivo | Vale para |
| --- | --- | --- |
| Global | `~/.codex/AGENTS.md` | Todas as sessões. O snippet se restringe a `~/Desktop/Convenia/` pela linha "Escopo". |
| Projeto específico | `<projeto>/AGENTS.md` | Só aquele projeto. |

Passos:

1. Abrir o arquivo do escopo escolhido. Criar se não existir.
2. Colar o snippet no fim do arquivo, sem apagar o conteúdo existente.
3. Abrir uma sessão nova dentro do projeto e perguntar ao agente qual referência de estilo frontend ele usa.

Restrição: o Codex procura `AGENTS.md` da raiz do repositório git até o diretório atual. Ele não lê `~/Desktop/Convenia/AGENTS.md`, porque essa pasta fica acima da raiz dos repos. Para cobrir todos os projetos Convenia no Codex, use o escopo global.

### Falhas comuns

- Agente não segue o vault: sessão aberta antes da instalação. Abrir sessão nova.
- Agente não acha os arquivos: caminho do vault diferente de `~/Desktop/Convenia/obsidian-frontend`. Corrigir os caminhos no snippet.
- Instruções duplicadas: snippet colado em global e em pasta/projeto ao mesmo tempo. Manter um escopo só.

## Política de atualização (beta)

- Correções de estilo dadas a qualquer agente durante o trabalho em projetos Convenia SHOULD virar ajuste no vault.
- O agente propõe o ajuste. O usuário autoriza. Só depois o agente edita.
- Toda edição segue [[AGENTS]] e confere se outras notas ficaram contraditórias.

## Referências cruzadas

- [[home]]
- [[AGENTS]]
- [[Playbook/playbook]]
- [[Playbook/red-flags]]
- [[Templates/templates]]
