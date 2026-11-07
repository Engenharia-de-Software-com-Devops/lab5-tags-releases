# Laboratório 5 — Tags, releases e histórico

Blog Hugo publicado em repositório remoto, com tag de versão `v1.0` e prática das operações de recuperação de histórico `git reset --soft` e `git revert`.

> Log bruto de todos os comandos executados e suas saídas: [`docs/execucao-completa.log`](docs/execucao-completa.log)

## Sumário

1. [Objetivo](#1-objetivo)
2. [Estrutura do repositório](#2-estrutura-do-repositório)
3. [Preparação do blog](#3-preparação-do-blog)
4. [Passo a passo executado](#4-passo-a-passo-executado)
5. [Validação final](#5-validação-final)
6. [Critérios de conclusão](#6-critérios-de-conclusão)
7. [Conceitos aprendidos](#7-conceitos-aprendidos)
8. [Observações](#8-observações)

## 1. Objetivo

- Conectar um repositório local a um repositório remoto (`git remote add`)
- Enviar a branch `main` (`git push -u`)
- Clonar o repositório numa segunda cópia local (`git clone`)
- Criar e publicar uma tag anotada (`git tag -a` + `git push origin <tag>`)
- Desfazer commits com `git reset --soft` (reescreve histórico local) e `git revert` (cria commit inverso)

## 2. Estrutura do repositório

```
lab5-tags-releases/
├── hugo.toml                  # configuração do site Hugo
├── archetypes/default.md      # modelo usado por `hugo new`
├── layouts/all.html           # layout único do site (substitui um tema externo)
├── content/posts/
│   ├── hello-world.md         # post inicial (vindo do Laboratório 1)
│   ├── hobbies.md             # post de hobbies, usado no exercício de reset/revert
│   └── perfil-profissional.md # post de perfil profissional
├── docs/execucao-completa.log # saída real de todos os comandos
└── README.md
```

O blog reaproveita o projeto do [Laboratório 1 (git-workshop)](https://github.com/Engenharia-de-Software-com-Devops/git-workshop), que usa um layout próprio em `layouts/all.html` em vez do tema `ananke` citado no enunciado. O resultado é o mesmo: um site Hugo funcional.

## 3. Preparação do blog

Repositório local criado com três commits, um por post:

```bash
git init -b main
git commit -m "feat: add Hugo blog base with hello-world post"
git commit -m "feat: add hobbies post"
git commit -m "feat: add professional profile post"
```

```
d932135 feat: add professional profile post
4da604b feat: add hobbies post
95c6fb0 feat: add Hugo blog base with hello-world post
```

## 4. Passo a passo executado

### 4.1 Repositório remoto

Criado na organização `Engenharia-de-Software-com-Devops` como **público** (o enunciado sugere privado; aqui é público para ficar visível no portfólio), sem README, `.gitignore` ou licença, para não haver histórico conflitante com o local.

### 4.2 Conectar o blog ao remoto

`git remote add` registra um apelido (`origin`) para a URL do remoto.

```bash
$ git remote add origin https://github.com/Engenharia-de-Software-com-Devops/lab5-tags-releases.git
$ git remote -v
origin	https://github.com/Engenharia-de-Software-com-Devops/lab5-tags-releases.git (fetch)
origin	https://github.com/Engenharia-de-Software-com-Devops/lab5-tags-releases.git (push)
```

**Resultado:** o mesmo endereço é usado para buscar (`fetch`) e enviar (`push`).

### 4.3 Push da branch main

`-u` grava `origin/main` como *upstream*, permitindo usar `git push`/`git pull` sem argumentos depois.

```bash
$ git push -u origin main
 * [new branch]      main -> main
branch 'main' set up to track 'origin/main'.

$ git branch -vv
* main d932135 [origin/main] feat: add professional profile post
```

**Resultado:** `[origin/main]` confirma o rastreamento da branch remota.

### 4.4 Clonar o repositório

```bash
$ cd .. && git clone https://github.com/Engenharia-de-Software-com-Devops/lab5-tags-releases.git blog-clonado
$ cd blog-clonado && ls
archetypes  content  hugo.toml  layouts

$ git branch --all
* main
  remotes/origin/HEAD -> origin/main
  remotes/origin/main
```

**Resultado:** cópia completa criada; o `clone` já configura o `origin` sozinho, sem `git remote add`.

### 4.5 Criar tag anotada v1.0

Tag **anotada** (`-a`) é um objeto próprio no Git, com autor, data e mensagem — diferente da tag leve, que é só um ponteiro.

```bash
$ cd ../meu-blog
$ git tag -a v1.0 -m "Primeira versão publicada do blog"
$ git tag
v1.0

$ git show v1.0 --stat
tag v1.0
Tagger: jaimedsf <jaime.silva.ferreira2@gmail.com>
Date:   Fri Nov 6 21:05:46 2026 -0300

Primeira versão publicada do blog

commit d932135845267a735b3f4c20c383b6ea66669617
    feat: add professional profile post
```

**Resultado:** `v1.0` aponta para o commit `d932135`, com metadados do autor da tag.

### 4.6 Push da tag

Tags não sobem com `git push` comum; precisam ser enviadas explicitamente.

```bash
$ git push origin v1.0
 * [new tag]         v1.0 -> v1.0

$ git ls-remote --tags origin
5d745a2aca67c4f04976f03d3cc3cb6abcd9148d	refs/tags/v1.0
d932135845267a735b3f4c20c383b6ea66669617	refs/tags/v1.0^{}
```

**Resultado:** a tag existe no remoto. A primeira linha é o objeto da tag anotada; `^{}` mostra o commit para o qual ela aponta.

### 4.7 git reset --soft

Branch de teste criada para não afetar a `main`. Linha adicionada na seção Esportes de `content/posts/hobbies.md`:

```diff
 Gosto de jogar futebol nos fins de semana.
+Também gosto de jogar basquete.
```

```bash
$ git switch -c test-reset
$ git add content/posts/hobbies.md
$ git commit -m "feat: adiciona basquete aos hobbies"
[test-reset 5c66d7f] feat: adiciona basquete aos hobbies

$ git reset --soft HEAD~1
$ git status
No ramo test-reset
Mudanças a serem submetidas:
	modified:   content/posts/hobbies.md

$ git commit -m "feat: adiciona basquete à seção de esportes"
[test-reset daa99e9] feat: adiciona basquete à seção de esportes
```

**O que aconteceu:** `--soft` moveu o ponteiro da branch um commit para trás, mas manteve a alteração no *staging*. Foi possível refazer o commit com mensagem melhor. O commit `5c66d7f` saiu do histórico da branch e foi substituído por `daa99e9`.

### 4.8 git revert

```bash
$ git revert --no-edit HEAD
[test-reset ddb77f6] Revert "feat: adiciona basquete à seção de esportes"
 1 file changed, 1 deletion(-)

$ git log --oneline
ddb77f6 Revert "feat: adiciona basquete à seção de esportes"
daa99e9 feat: adiciona basquete à seção de esportes
d932135 feat: add professional profile post
4da604b feat: add hobbies post
95c6fb0 feat: add Hugo blog base with hello-world post
```

**O que aconteceu:** o `revert` não apagou nada; criou um **novo commit** (`ddb77f6`) que aplica o inverso de `daa99e9`. O histórico fica preservado, por isso é a forma segura de desfazer algo já publicado. (`--no-edit` aceita a mensagem padrão sem abrir o editor.)

### 4.9 Excluir a branch de teste

```bash
$ git switch main
$ git branch -d test-reset
error: the branch 'test-reset' is not fully merged

$ git branch -D test-reset
Deleted branch test-reset (was ddb77f6).
```

**O que aconteceu:** `-d` recusou porque `test-reset` tinha commits fora da `main` (proteção contra perda de trabalho). `-D` força a exclusão. Aqui foi intencional, já que a branch era só para treino.

## 5. Validação final

```bash
$ git status
No ramo main
Your branch is up to date with 'origin/main'.
nothing to commit, working tree clean

$ git log --oneline
d932135 feat: add professional profile post
4da604b feat: add hobbies post
95c6fb0 feat: add Hugo blog base with hello-world post

$ git tag
v1.0

$ git branch
* main
```

No GitHub: branch `main` com os commits e a tag `v1.0` visível em **Tags**.

> Depois da validação, este README e o log foram adicionados num commit posterior na `main`. A tag `v1.0` continua apontando para `d932135`, a versão do blog marcada no exercício.

## 6. Critérios de conclusão

- [x] `git remote -v` mostra a URL do repositório remoto
- [x] `git branch -vv` mostra `[origin/main]` como upstream
- [x] `blog-clonado` existe como cópia local completa do blog
- [x] A tag `v1.0` existe localmente e no repositório remoto
- [x] O `git reset --soft` desfez o commit mantendo as alterações staged
- [x] O `git revert` criou um novo commit desfazendo as alterações
- [x] A branch `test-reset` foi excluída
- [x] `git status` está limpo

## 7. Conceitos aprendidos

| Comando | O que faz | Reescreve histórico? | Quando usar |
|---|---|---|---|
| `git remote add` | Registra um apelido para uma URL remota | Não | Ligar repositório local a um servidor |
| `git push -u` | Envia commits e define upstream | Não | Primeiro envio de uma branch |
| `git clone` | Copia repositório completo e configura `origin` | Não | Nova cópia de trabalho |
| `git tag -a` | Cria tag anotada (com autor, data e mensagem) | Não | Marcar versões/releases |
| `git push origin <tag>` | Publica uma tag | Não | Tags não sobem sozinhas |
| `git reset --soft` | Volta o ponteiro da branch mantendo mudanças staged | **Sim** | Corrigir commits ainda **não publicados** |
| `git revert` | Cria commit inverso | Não | Desfazer commits **já publicados** |
| `git branch -D` | Exclui branch mesmo sem merge | Remove referência | Descartar trabalho de teste |

## 8. Observações

- O enunciado pede repositório privado; este é público por estar na organização da disciplina.
- O tema `ananke` foi substituído pelo layout próprio herdado do Laboratório 1.
- O `git revert` foi executado com `--no-edit` para não precisar abrir o editor; o resultado é o mesmo.
- Ambiente: Git em Linux, autenticação HTTPS via GitHub CLI.

## Referências

- [git-remote](https://git-scm.com/docs/git-remote) · [git-push](https://git-scm.com/docs/git-push) · [git-clone](https://git-scm.com/docs/git-clone)
- [git-tag](https://git-scm.com/docs/git-tag) · [git-reset](https://git-scm.com/docs/git-reset) · [git-revert](https://git-scm.com/docs/git-revert)
- [Learning Git (caps. 6–8)](https://gitlearningjourney.com/learning-git)
