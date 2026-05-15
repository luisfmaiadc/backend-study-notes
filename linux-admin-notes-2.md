# Shell e Navegação Básica

## Shell e Bash

**Shell** é o interpretador de comandos — a interface entre o usuário e o kernel. Ele recebe os comandos digitados e os repassa ao sistema operacional para execução.

**Bash** (Bourne Again Shell) é o shell padrão na maioria das distribuições Linux. É uma evolução do shell original `sh` (Bourne Shell).

---

## Anatomia do Prompt

```
root@linux-admin:~#
```

| Elemento | Significado |
|---|---|
| `root` | Nome do usuário da sessão |
| `@` | "Na" (indica onde o usuário está) |
| `linux-admin` | Nome da máquina (hostname) |
| `~` | Diretório home do usuário atual |
| `#` | Usuário é administrador (root) |
| `$` | Usuário comum (sem privilégios de admin) |

> O usuário **root** (admin) pode realizar qualquer operação na máquina, sem restrições.

---

## Diretórios Especiais

| Símbolo | Referência |
|---|---|
| `~` | Diretório home do usuário atual |
| `.` | Diretório atual |
| `..` | Diretório pai (anterior) |
| `/` | Raiz do sistema (equivalente ao `C:\` no Windows) |
| `-` | Último diretório visitado |

---

## Comandos

### Sessão

| Comando | Descrição |
|---|---|
| `exit` / `logout` / `Ctrl+D` | Encerra a sessão atual |
| `su -` | Alterna para o usuário root |
| `clear` / `Ctrl+L` | Limpa a tela do terminal |
| `history` | Exibe o histórico de comandos recentes |

### Navegação

| Comando | Descrição |
|---|---|
| `pwd` | Exibe o diretório atual (*Print Working Directory*) |
| `cd <dir>` | Navega para o diretório informado |
| `cd` | Navega diretamente para o home do usuário |
| `cd -` | Volta ao último diretório visitado |
| `cd ..` | Sobe um nível na hierarquia de diretórios |
| `cd ../../..` | Sobe múltiplos níveis (neste exemplo, 3) |

### Arquivos e Listagem

| Comando | Descrição |
|---|---|
| `cat <arquivo>` | Exibe o conteúdo de um arquivo |
| `ls` | Lista arquivos e diretórios |
| `ls -a` | Lista incluindo arquivos ocultos (iniciados por `.`) |
| `ls -lha` | Lista com detalhes e tamanhos legíveis por humanos |

> Parâmetros podem ser combinados e em qualquer ordem: `ls -lha` e `ls -alh` produzem o mesmo resultado.

### Busca e Disco

| Comando | Descrição |
|---|---|
| `grep <termo> <arquivo>` | Busca um termo dentro de um arquivo ou saída |
| `df -h` | Exibe partições do sistema com tamanhos legíveis |

---

## Dicas do Terminal

- **`Tab`** — autocompleta comandos, caminhos e nomes de arquivos
- **`Tab` 2x** — exibe todas as opções disponíveis para completar o comando atual
- Arquivos e diretórios iniciados por `.` são **ocultos** e não aparecem em um `ls` simples