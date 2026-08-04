# Gerenciamento de Processos

## $PATH

`$PATH` é uma **variável de ambiente** que contém uma lista de diretórios onde o shell busca executáveis. Quando um comando é digitado sem o caminho completo, o shell percorre esses diretórios na ordem definida até encontrar o binário correspondente.

```bash
echo $PATH
# Saída exemplo: /usr/local/sbin:/usr/local/bin:/usr/sbin:/usr/bin:/sbin:/bin
```

> **Segurança:** ao elevar para sessão de superusuário, prefira usar o caminho completo — `/bin/su` em vez de apenas `su` — para evitar que um binário malicioso com o mesmo nome, presente em outro diretório do `$PATH`, seja executado no lugar do original.

---

## Execução de Múltiplos Comandos

| Operador | Comportamento |
|---|---|
| `;` | Executa os comandos **sequencialmente**, um após o outro |
| `\|` (pipe) | Encadeia comandos — a **saída** do primeiro é usada como **entrada** do segundo |

```bash
ls ; tree ; sleep 1          # Executa ls, depois tree, depois sleep
ls -a | less                 # Passa a saída do ls como entrada para o less
```

---

## Processos em Primeiro e Segundo Plano

Processos podem ser executados em **primeiro plano** (bloqueando o terminal até concluir) ou em **segundo plano** (liberando o terminal enquanto o processo continua).

### Enviar para segundo plano ao iniciar

```bash
comando &    # O & ao final envia o processo direto para segundo plano
```

### Mover entre planos durante a execução

| Ação | Como fazer |
|---|---|
| Pausar processo em primeiro plano | `Ctrl+Z` — suspende o processo |
| Retomar em segundo plano | `bg n` — retoma o processo suspenso em background (n = índice exibido pelo `Ctrl+Z`) |
| Trazer de volta ao primeiro plano | `fg n` — traz o processo de volta ao terminal (n = índice do job) |

### Finalizar um processo em segundo plano

| Opção | Comandos |
|---|---|
| Via primeiro plano | `fg n` → `Ctrl+C` |
| Direto pelo índice | `kill %n` (n = índice do job) |

---

## Sinais do Sistema

Sinais são o mecanismo de **comunicação entre processos** (IPC) no Linux. Permitem que o kernel ou outros processos notifiquem um processo sobre eventos específicos.

| Sinal | Número | Descrição |
|---|---|---|
| `SIGHUP` | `1` | Hangup — enviado quando o terminal é fechado |
| `SIGINT` | `2` | Interrupt — enviado pelo `Ctrl+C` |
| `SIGTERM` | `15` | Solicitação amigável de encerramento (padrão do `kill`) |
| `SIGKILL` | `9` | Encerramento forçado — não pode ser ignorado ou interceptado pelo processo |

---

## Listagem de Processos — `ps`

| Comando | Descrição |
|---|---|
| `ps` | Lista processos do **terminal atual** |
| `ps -a` | Lista processos ativos em **outros terminais** |
| `ps -ax` | Lista **todos os processos** do sistema, de todos os usuários |
| `ps -axu` | Igual ao anterior, incluindo o **usuário** associado a cada processo |
| `ps -axm` | Inclui detalhes de **uso de memória** por processo |
| `ps -axf` | Exibe os processos em formato de **árvore** (quais processos chamam quais) |
| `ps -axe` | Inclui as **variáveis de ambiente** utilizadas por cada processo |
| `ps -axew` | Igual ao anterior, mas sem cortar linhas longas — variáveis extensas continuam na linha abaixo |
| `ps --sort=pid` | Classifica a listagem por **PID** (pode-se usar outras colunas como `%cpu`, `%mem`, etc.) |

### Coluna STAT — Status do processo

| Código | Significado |
|---|---|
| `R` | **Running** — em execução ou na fila de execução |
| `S` | **Sleeping** — aguardando um evento (interrompível) |
| `D` | **Uninterruptible sleep** — aguardando I/O, não pode ser interrompido (nem pelo kill -9) |
| `Z` | **Zombie** — processo encerrado, mas não coletado pelo processo pai |
| `X` | **Dead** — processo morto (raramente visível) |
| `I` | **Idle** — thread do kernel inativa |
| `<` | Alta prioridade (nice negativo) |
| `N` | Baixa prioridade (nice positivo) |
| `s` | Líder de sessão |
| `l` | Multi-thread |
| `+` | Em execução no grupo de primeiro plano |

---

## Controle de Processos

### `jobs` — Listar processos em segundo plano

```bash
jobs    # Exibe todos os processos em execução em segundo plano e seus índices
```

### `kill` — Encerrar processos

| Comando | Descrição |
|---|---|
| `kill %n` | Encerra o job de índice `n` (listado pelo `jobs`) de forma amigável |
| `kill <PID>` | Envia `SIGTERM` (-15) ao processo — pede encerramento amigável |
| `kill -9 <PID>` | Envia `SIGKILL` — encerramento **forçado**, remove o processo da memória imediatamente |
| `kill -HUP <PID>` | Envia `SIGHUP` (-1) — faz o processo **reler seus arquivos de configuração** sem encerrar |

### `pidof` — Obter PID de um processo

| Comando | Descrição |
|---|---|
| `pidof <processo>` | Retorna o(s) PID(s) de todos os processos com o nome especificado |
| `pidof -s <processo>` | Retorna apenas o PID do **primeiro** processo encontrado |

### `pgrep` e `pkill` — Gerenciar processos pelo nome

| Comando | Descrição |
|---|---|
| `pgrep -u root sshd` | Lista os PIDs de processos cujo nome corresponde ao padrão, filtrando por usuário |
| `pkill -u root sleep` | Encerra processos pelo nome e critérios informados (equivalente ao `killall`) |

### `killall5` — Encerrar todos os processos

```bash
killall5    # Envia um sinal de encerramento para TODOS os processos do sistema
```

> Usado em scripts de desligamento e runlevels. Diferente do `pkill`, não filtra por nome.

### `nohup` — Proteger processo de sinais de interrupção

```bash
nohup <comando> &
```

`nohup` faz o processo ignorar o sinal `SIGHUP` — que normalmente é enviado quando o terminal é fechado. Com isso, o processo continua em execução mesmo após a sessão ser encerrada. A saída é redirecionada automaticamente para o arquivo `nohup.out`.

---

## Árvore de Processos — `pstree`

| Comando | Descrição |
|---|---|
| `pstree` | Exibe todos os processos em forma de **árvore hierárquica** |
| `pstree -c` | Exibe também os **processos pai** de forma expandida |
| `pstree -h` | **Destaca** o processo atual e seus ancestrais na árvore |
| `pstree -p` | Exibe o **PID** de cada processo na árvore |
| `pstree -H <PID>` | Exibe e destaca apenas a árvore do **processo informado** |
| `pstree -u` | Exibe as **mudanças de usuário** na árvore (quando um processo filho roda como usuário diferente do pai) |
| `pstree -g` | Exibe as **mudanças de grupo** na árvore, de forma semelhante ao `-u` |

---

## Monitoramento de Processos — `top`

`top` exibe em tempo real os processos em execução, com dados de CPU, memória, carga do sistema e mais — atualizando automaticamente a cada **2 segundos**.

```bash
top
top -i    # Ignora processos inativos
top -c    # Exibe a linha de comando completa de cada processo (com parâmetros)
```

### Atalhos dentro do `top`

| Tecla | Ação |
|---|---|
| `1` | Alterna exibição detalhada por **núcleo de CPU** |
| `m` | Alterna exibição detalhada de **memória** |
| `Shift + M` | Ordena processos por **uso de memória** |
| `Shift + P` | Ordena processos por **uso de CPU** |
| `Shift + <coluna>` | Ordena pelos dados da coluna escolhida |
| `f` | Abre o seletor de **colunas** disponíveis para exibição e ordenação |
| `k` | Solicita um **PID** para encerrar o processo (kill) sem sair do top |
| `d` | Altera o **intervalo de atualização** da exibição |
| `Shift + W` | **Salva** a configuração atual do top no diretório home do usuário |

> Sem salvar com `Shift+W`, o top volta à exibição padrão toda vez que for aberto novamente.

---

## Prioridade de Processos

### `nice` — Definir prioridade ao iniciar

Prioridades vão de **-20** (maior prioridade) a **19** (menor prioridade). O padrão é `0`. Apenas o root pode definir valores negativos.

```bash
nice -n <prioridade> <comando>

nice -n -10 processo    # Alta prioridade (root apenas)
nice -n 15 processo     # Baixa prioridade (qualquer usuário)
```

### `renice` — Ajustar prioridade de processo em execução

```bash
renice -n <prioridade> -p <PID>
```

---

## Estatísticas do Sistema

### `tload` — Gráfico de carga do sistema

`tload` exibe um **gráfico ASCII em tempo real** da carga média do sistema (load average) diretamente no terminal — uma representação visual do mesmo indicador mostrado por `uptime` e `top`.

### `vmstat` — Estatísticas de memória e sistema

`vmstat` exibe em formato tabular o estado do sistema: processos, memória, swap, I/O de disco, interrupções e uso de CPU.

| Comando | Descrição |
|---|---|
| `vmstat` | Exibe uma linha de estatísticas no momento atual |
| `vmstat N` | Atualiza e exibe as estatísticas a cada **N segundos** |

> Colunas principais: `r` (processos em fila), `b` (processos bloqueados), `free` (memória livre), `si/so` (swap in/out), `us/sy/id/wa` (CPU: usuário, sistema, idle, aguardando I/O).