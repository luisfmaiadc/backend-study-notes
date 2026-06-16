# Manipulação de Arquivos e Diretórios

## Gerenciamento de Energia

| Comando | Descrição |
|---|---|
| `halt` / `poweroff` / `init 0` | Desliga a máquina |
| `reboot` / `init 6` | Reinicia a máquina |
| `shutdown -h now` | Desliga imediatamente |
| `shutdown -r now` | Reinicia imediatamente |
| `shutdown -h 18:00` | Agenda desligamento para um horário específico |
| `shutdown -c` | Cancela um desligamento agendado |

---

## Comandos Internos e Externos

Comandos no Linux podem ser **internos** (built-ins do shell, executados diretamente pelo Bash) ou **externos** (programas armazenados no sistema de arquivos). Comandos internos são executados de forma significativamente mais rápida.

Quando existem dois comandos com o mesmo nome — um interno e um externo — o Linux **prioriza o interno**. Para forçar a execução do externo, é necessário informar o caminho completo do executável.

| Comando | Descrição |
|---|---|
| `type <comando>` | Informa diretamente se o comando é interno ou externo |
| `which <comando>` | Exibe o caminho do executável — se apontar para `/bin`, `/usr/bin`, etc., é externo |
| `man builtins` | Lista todos os comandos internos do Bash |

---

## Listagem — `ls`

`ls` lista o conteúdo de um diretório em ordem alfabética. Aceita parâmetros curtos combinados (ex: `ls -lha`) ou extensos separados (ex: `ls -l --color=auto`).

| Comando | Descrição |
|---|---|
| `ls` | Lista arquivos e diretórios do diretório atual |
| `ls <dir>` | Lista o conteúdo do diretório informado |
| `ls <dir1> <dir2>` | Lista o conteúdo de múltiplos diretórios de uma vez |
| `ls -a` | Inclui arquivos ocultos, `.` (atual) e `..` (anterior) |
| `ls -A` | Igual ao `-a`, mas omite `.` e `..` |
| `ls -B` | Omite arquivos de backup (terminados em `~`) |
| `ls -l` | Exibe detalhes: permissões, dono, grupo, tamanho, data e nome |
| `ls -lh` | Igual ao `-l`, mas com tamanhos legíveis (KB, MB, GB) |
| `ls -d` | Exibe informações do próprio diretório, não do seu conteúdo — útil com `-l` |
| `ls -f` | Lista como `-a`, mas ordenando pelo tempo da última alteração |
| `ls -F` | Igual ao `-f`, mas adiciona sinalizadores ao nome: `/` para diretórios, `@` para links simbólicos, `*` para executáveis |
| `ls -l -L` | Ao encontrar links simbólicos, exibe o tamanho do arquivo real apontado |
| `ls -t` | Ordena pelo mais recente primeiro |
| `ls -tr` | Ordena do mais antigo ao mais recente |
| `ls -R` | Lista recursivamente todo o conteúdo de subdiretórios |
| `ls --color=auto` | Diferencia tipos de arquivo por cores (diretórios em azul, arquivos em branco, etc.) |

### Exemplo de saída — `ls -l`

```
drwxr-xr-x 5 root root 4096 abr 29 18:14 grub
│           │ │    │    │    │            └─ Nome
│           │ │    │    │    └─ Data de criação/modificação
│           │ │    │    └─ Tamanho
│           │ │    └─ Grupo
│           │ └─ Dono
│           └─ Número de links
└─ Permissões (tipo + dono + grupo + outros)
```

---

## Criação de Diretórios — `mkdir`

| Comando | Descrição |
|---|---|
| `mkdir <dir>` | Cria um novo diretório |
| `mkdir -p <dir/subdir/subsubdir>` | Cria toda a hierarquia de diretórios de uma vez |

---

## Remoção de Diretórios Vazios — `rmdir`

| Comando | Descrição |
|---|---|
| `rmdir <dir>` | Remove um diretório **vazio** |
| `rmdir -p <dir/subdir>` | Remove a hierarquia de diretórios vazios de uma vez |

---

## Remoção de Arquivos e Diretórios — `rm`

> **Atenção:** o `rm` não move para a lixeira — a exclusão é permanente.

| Comando | Descrição |
|---|---|
| `rm <arquivo>` | Remove um arquivo |
| `rm -r <dir>` | Remove um diretório e seu conteúdo recursivamente |
| `rm -f <arquivo>` | Força a remoção sem solicitar confirmação |
| `rm -rf <dir>` | Remove recursivamente diretórios e arquivos sem confirmação |
| `rm -rf *` | Remove **todo** o conteúdo não oculto do diretório atual |
| `rm -f bb*` | Remove todos os arquivos cujo nome começa com `bb` |

> `*` é um **caractere curinga** — representa qualquer sequência de caracteres. Pode ser usado em outros comandos além do `rm`.

---

## Visualização de Arquivos — `cat`, `tac`, `zcat`

| Comando | Descrição |
|---|---|
| `cat <arquivo>` | Exibe o conteúdo de um arquivo |
| `cat -n <arquivo>` | Exibe o conteúdo numerando todas as linhas |
| `cat -b <arquivo>` | Numera apenas as linhas com conteúdo (ignora linhas em branco) |
| `cat -s <arquivo>` | Suprime linhas em branco repetidas consecutivas |
| `cat -E <arquivo>` | Marca o fim de cada linha com `$` — útil para depuração |
| `tac <arquivo>` | Exibe o conteúdo do arquivo de baixo para cima (inverso do `cat`) |
| `zcat <arquivo.gz>` | Exibe o conteúdo de arquivos compactados sem descompactá-los |

---

## Cópia de Arquivos e Diretórios — `cp`

| Comando | Descrição |
|---|---|
| `cp <origem> <destino>` | Copia um arquivo ou diretório para o destino |
| `cp <origem> <novoNome>` | Copia o arquivo com um novo nome dentro do mesmo diretório |
| `cp * <destino>` | Copia todo o conteúdo do diretório atual para o destino |
| `cp -v <origem> <destino>` | Modo verbose — exibe cada arquivo sendo copiado |
| `cp -vs <origem> <destino>` | Cria um link simbólico (atalho) em vez de copiar o arquivo |
| `cp -u <origem> <destino>` | Copia apenas se o arquivo não existir no destino ou se a origem for mais recente |
| `cp -p <origem> <destino>` | Preserva os metadados do arquivo (datas, permissões) |
| `cp -a <origem> <destino>` | Equivalente a `-d -R -p`: preserva links simbólicos, copia recursivamente e mantém metadados |

> Ao copiar múltiplos arquivos em um único comando, o destino deve obrigatoriamente ser um diretório válido.

---

## Movimentação e Renomeação — `mv`

| Comando | Descrição |
|---|---|
| `mv <origem> <destino>` | Move o arquivo para o destino, removendo-o da origem |
| `mv <nome> <novoNome>` | Renomeia o arquivo no mesmo diretório |
| `mv -i <origem> <destino>` | Solicita confirmação antes de sobrescrever arquivo existente no destino |
| `mv -f <origem> <destino>` | Força a sobrescrita sem solicitar confirmação |
| `mv -v <origem> <destino>` | Modo verbose — exibe cada arquivo sendo movido |
| `mv -u <origem> <destino>` | Move apenas se a origem for mais recente que o arquivo no destino |

---

## Visualização de Estrutura — `tree`

| Comando | Descrição |
|---|---|
| `tree` | Exibe a estrutura de diretórios atual de forma hierárquica e visual |