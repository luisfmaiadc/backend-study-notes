# Editores de Texto (Micro Tutorial)

> Este é um panorama introdutório dos principais editores de texto em modo terminal, o conteúdo será aprofundado posteriormente.

## nano

O **nano** é o mais simples dos editores que acompanham o Debian. Ideal para edições rápidas e diretas.

```
nano <caminhoDoArquivo>
```

> `^` nos atalhos do nano representa a tecla **Control (Ctrl)**.

| Atalho | Descrição |
|---|---|
| `Ctrl + O` | Salva as modificações (*write Out*) |
| `Ctrl + X` | Salva e sai do editor |

> Os principais comandos disponíveis ficam sempre listados na parte inferior do terminal.

---

## mcedit

O **mcedit** é o editor de texto que acompanha o pacote `mc` (Midnight Commander). É mais poderoso que o nano, com funções acionadas pelas teclas de função (`F1`, `F2`, ...).

| Atalho | Descrição |
|---|---|
| `F10` ou `Esc` (2x) | Sai do editor — pergunta se deseja salvar, caso haja modificações pendentes |

> Assim como no nano, as funções disponíveis aparecem na parte inferior do terminal.

---

## vi

O **vi** é um editor tradicional e poderoso, porém mais complexo por ser um **editor modal** — ou seja, possui modos distintos de operação.

### Modos do vi

| Modo | Descrição |
|---|---|
| **Normal** (padrão) | Modo inicial ao abrir o arquivo — não permite edição direta de texto, apenas navegação e comandos |
| **Inserção** | Permite editar o texto livremente |
| **Comando** | Acessado via barra de status, para executar ações como salvar e sair |

### Navegando entre os modos

| Tecla | Ação |
|---|---|
| `i` | Entra no modo de inserção (edição) |
| `Esc` | Retorna ao modo normal |
| `Shift + :` | Abre a barra de status para digitar comandos |

### Comandos (via barra de status)

| Comando | Descrição |
|---|---|
| `:w` | Grava (salva) sem sair do editor |
| `:x` | Salva e sai do editor |
| `:q` | Sai do editor (somente se não houver modificações pendentes) |
| `:q!` | Sai sem salvar, descartando todas as modificações realizadas |