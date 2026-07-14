# Comandos Gerais — Linux

## Comentários no Shell

Prefixar uma linha com `#` faz com que o shell a trate como **comentário** — o comando não é executado. Ainda assim, aparece no histórico exatamente como digitado, o que é útil para anotar e documentar ações no terminal.

```bash
# Isso é um comentário e não será executado
```

---

## Visualização de Conteúdo

| Comando | Descrição |
|---|---|
| `more <arquivo>` | Exibe o arquivo de forma **paginada** — avança com `Space`, sai com `q`. Não permite rolar para cima |
| `less <arquivo>` | Exibe o arquivo com rolagem para cima e para baixo, e navegação lateral. Mais completo que o `more` |
| `head <arquivo>` | Exibe as **10 primeiras** linhas do arquivo |
| `head -n N <arquivo>` | Exibe as **N primeiras** linhas do arquivo |
| `tail <arquivo>` | Exibe as **10 últimas** linhas do arquivo |
| `tail -n N <arquivo>` | Exibe as **N últimas** linhas do arquivo |
| `tail -f <arquivo>` | Monitora o arquivo **em tempo real**, exibindo novas linhas à medida que são adicionadas — muito útil para acompanhar logs |
| `nl <arquivo>` | Exibe o conteúdo do arquivo com **linhas numeradas** |

### Parâmetros do `nl`

| Parâmetro | Descrição |
|---|---|
| `-b a` | Numera **todas** as linhas, incluindo as em branco |
| `-b t` | Numera apenas as linhas **com conteúdo** (padrão) |
| `-n ln` | Alinha os números à **esquerda** |
| `-n rn` | Alinha os números à **direita** (padrão) |
| `-n rz` | Alinha à direita e preenche com **zeros** à esquerda |
| `-v N` | Define o **número inicial** da contagem |

### Pesquisa dentro do `less`

Durante a execução do `less`, é possível fazer buscas no conteúdo pressionando `/` seguido do termo desejado — funciona como um `Ctrl+F`.

---

## Ordenação e Contagem

### `sort` — Ordenação de conteúdo

| Comando | Descrição |
|---|---|
| `sort <arquivo>` | Ordena o conteúdo **alfabeticamente** (crescente) — trata tudo como texto |
| `sort -r <arquivo>` | Inverte a ordenação para **decrescente** |
| `sort -n <arquivo>` | Ordena **numericamente** de forma crescente |
| `sort -c <arquivo>` | Verifica se o conteúdo já está ordenado e exibe o resultado da avaliação |

### `wc` — Contagem de linhas, palavras e bytes

| Comando | Descrição |
|---|---|
| `wc <arquivo>` | Exibe quantidade de **linhas**, **palavras** e **bytes** do arquivo |
| `wc -l <arquivo>` | Exibe apenas a quantidade de **linhas** |
| `wc -w <arquivo>` | Exibe apenas a quantidade de **palavras** |
| `wc -c <arquivo>` | Exibe apenas a quantidade de **bytes** |

---

## Pesquisa em Conteúdo — `grep`

`grep` pesquisa um padrão dentro de arquivos ou da saída de outros comandos.

```bash
grep <expressao> <arquivo>
```

| Comando | Descrição |
|---|---|
| `grep 'root' /etc/passwd` | Exibe as linhas que contêm "root" |
| `grep -v 'root' /etc/passwd` | Inverte a busca — exibe tudo que **não** contém "root" |
| `grep -i <expressao> <arquivo>` | Ignora diferença entre maiúsculas e minúsculas (case insensitive) |
| `grep -f <arquivo_padroes> <arquivo>` | Lê os padrões de pesquisa a partir de um arquivo (um padrão por linha) |
| `grep -E <expressao> <arquivo>` | Habilita **expressões regulares estendidas** |
| `grep -iE 'www-data.*exemplo' <arquivo>` | Busca case insensitive com expressão regular — `.*` significa "qualquer coisa entre" |
| `grep -iF <expressao> <arquivo>` | Interpreta caracteres especiais como texto literal — útil quando o padrão contém `*`, `.`, etc. |
| `grep -ri <expressao> <dir>` | Pesquisa **recursivamente** em diretórios, com case insensitive |
| `grep -ril <expressao> <dir>` | Igual ao `-ri`, mas exibe apenas os **nomes dos arquivos** que contêm o padrão |
| `grep -rin <expressao> <dir>` | Igual ao `-ri`, mas também exibe o **número da linha** onde o padrão foi encontrado |

---

## Busca de Arquivos — `find`

`find` percorre a árvore de diretórios a partir de um ponto de partida e localiza arquivos ou diretórios conforme os critérios informados.

```bash
find <local> <argumento> <valor>
```

### Busca por nome e tipo

| Exemplo | Descrição |
|---|---|
| `find . -name cachorro` | Busca arquivos ou diretórios com o nome exato "cachorro" a partir do diretório atual |
| `find . -type d -name cachorro` | Busca apenas **diretórios** com esse nome |
| `find . -type f -name cachorro` | Busca apenas **arquivos** com esse nome |
| `find /usr -maxdepth 2 -type f -name cachorro` | Limita a busca a no máximo **2 níveis** de profundidade a partir do diretório informado |
| `find /usr -mindepth 2 -type f -name cachorro` | Inicia a busca somente a partir do **2º nível** de profundidade |

### Busca por data

O Linux registra três tipos de timestamps para cada arquivo:

| Tipo | Significado |
|---|---|
| **atime** *(access time)* | Último acesso ao conteúdo do arquivo |
| **mtime** *(modification time)* | Última modificação do conteúdo do arquivo |
| **ctime** *(change time)* | Última alteração nos metadados (permissões, dono) ou conteúdo |

> O sinal indica o intervalo: `-N` significa "nos últimos N", `+N` significa "há mais de N".

| Exemplo | Descrição |
|---|---|
| `find /etc -mtime -1` | Arquivos **modificados** há menos de 1 dia |
| `find /etc -amin -10` | Arquivos **acessados** nos últimos 10 minutos |
| `find /etc -cmin -1` | Arquivos com **metadados alterados** no último minuto |
| `find /etc -ctime +2` | Arquivos com **metadados alterados** há mais de 2 dias |

### Busca por dono e grupo

| Exemplo | Descrição |
|---|---|
| `find /home -user nomeUsuario` | Arquivos e diretórios cujo **dono** é o usuário especificado |
| `find /home -group nomeGrupo` | Arquivos e diretórios cujo **grupo** é o especificado |

### Outros parâmetros disponíveis

`-size`, `-links`, `-gid`, `-uid`, `-type b` (bloco), `-type c` (caractere), `-type s` (socket), `-type l` (link simbólico)

---

## Data e Hora

### `date` — Data e hora do sistema

| Comando | Descrição |
|---|---|
| `date` | Exibe a data e hora atuais do sistema |
| `date -u` | Exibe o horário **UTC/GMT** (fuso base Greenwich) |
| `date -s "DD/MM/YYYY HH:MM"` | Altera o horário do sistema manualmente |
| `date MMddhhmmyyyy` | Altera a data completa no formato mês, dia, hora, minuto e ano |
| `date +%d` | Exibe apenas o **dia** atual |
| `date +%m` | Exibe apenas o **mês** atual |
| `date +%Y` | Exibe o **ano** completo |
| `date +%y` | Exibe o **ano** abreviado (dois dígitos) |
| `date +%T` | Exibe a **hora** atual (HH:MM:SS) |
| `date +%j` | Exibe o **dia do ano** (1 a 365) |
| `date +%d-%m-%Y` | Exibe dia, mês e ano com separador |
| `date +"%d-%m-%Y %T"` | Exibe data e hora — aspas obrigatórias quando há espaço no formato |

### `hwclock` — Relógio de hardware

| Comando | Descrição |
|---|---|
| `hwclock --systohc` | Sincroniza o relógio da **placa-mãe** com o horário do sistema operacional |

---

## Disco e Armazenamento

### `df` — Uso das partições

| Comando | Descrição |
|---|---|
| `df` | Exibe informações das partições montadas |
| `df -h` | Exibe os dados em formato **legível** (KB, MB, GB — base 1024) |
| `df -l` | Exibe apenas sistemas de arquivos **locais** (exclui partições de rede) |
| `df -m` | Exibe os dados em **megabytes** |
| `df -T` | Exibe o **tipo do sistema de arquivos** de cada partição (ex: ext4, tmpfs) |
| `df -t ext4` | Exibe apenas partições formatadas com o **sistema de arquivos** especificado |

### `du` — Tamanho de diretórios e arquivos

| Comando | Descrição |
|---|---|
| `du` | Exibe o espaço ocupado por diretórios, subdiretórios e arquivos a partir do diretório atual |
| `du -h` | Exibe em formato legível com base em blocos de **1024** |
| `du -H` | Exibe em formato legível com base em **1000** (padrão SI) |
| `du -s` | Exibe apenas o **total** do diretório atual, sem detalhar o conteúdo |
| `du -hs` | Total em formato **legível** |
| `du -hc` | Exibe os detalhes e também a **soma total** ao final |
| `du -k` | Exibe em **kilobytes** |
| `du -m` | Exibe em **megabytes** |

### `ln` — Links

O Linux possui dois tipos de link:

| Tipo | Comportamento |
|---|---|
| **Hard link** | Aponta para o mesmo **inode** (mesmo dado físico em disco). Se o arquivo original for excluído, o dado continua acessível pelo link. Não funciona entre sistemas de arquivos diferentes |
| **Link simbólico** | É um **atalho** que referencia o arquivo original pelo caminho. Se o original for excluído, o link quebra. Pode apontar para arquivos em sistemas de arquivos diferentes |

| Comando | Descrição |
|---|---|
| `ln <arquivo> <nomeHardLink>` | Cria um **hard link** para o arquivo |
| `ln -s <arquivo> <nomeLinkSimbólico>` | Cria um **link simbólico** para o arquivo |

---

## Memória — `free`

| Comando | Descrição |
|---|---|
| `free` | Exibe uso e disponibilidade de **RAM** e **SWAP** |
| `free -kilo` | Exibe em **kilobytes** (base 1000) |
| `free -mega` | Exibe em **megabytes** (base 1000) |
| `free --kibi` | Exibe em **kibibytes** (base 1024) |
| `free --mebi` | Exibe em **mebibytes** (base 1024) |
| `free --gibi` | Exibe em **gibibytes** (base 1024) |
| `free --giga` | Exibe em **gigabytes** (base 1000) |
| `free -s 1` | Atualiza e exibe os resultados a **cada 1 segundo** |

---

## Informações do Sistema

### `uname` — Informações do kernel e sistema

| Comando | Descrição |
|---|---|
| `uname` | Exibe o nome do sistema operacional |
| `uname -a` | Exibe todas as informações: SO, hostname, versão do kernel, arquitetura, etc. |
| `uname -s` | Nome do kernel |
| `uname -n` / `hostname` | Nome da máquina na rede (hostname) |
| `uname -r` | Versão (release) do kernel |
| `uname -v` | Data e hora de compilação do kernel |
| `uname -m` | Arquitetura da máquina (ex: x86_64) |

### `uptime` — Tempo de atividade

| Comando | Descrição |
|---|---|
| `uptime` | Exibe há quanto tempo o sistema está em execução desde o último boot, além da carga média |

### `dmesg` — Mensagens do kernel

`dmesg` exibe as mensagens do **ring buffer** do kernel — registros de eventos de hardware, drivers e inicialização do sistema. É útil para diagnosticar problemas com dispositivos e verificar o que ocorreu durante o boot.

| Comando | Descrição |
|---|---|
| `dmesg` | Exibe todas as mensagens do ring buffer do kernel |
| `dmesg -t` | Exibe as mensagens **sem o timestamp** |
| `dmesg -T` | Exibe as mensagens com o timestamp em **formato legível** (data e hora) |
| `dmesg -x` | Exibe o **nível de log** de cada mensagem (info, warning, error, etc.) |
| `dmesg -c` | Exibe as mensagens e **limpa o ring buffer** após a leitura |

---

## Utilitários Gerais

### `echo` — Exibir texto na tela

| Comando | Descrição |
|---|---|
| `echo "mensagem"` | Exibe o texto na tela com quebra de linha ao final |
| `echo -n "mensagem"` | Exibe o texto **sem** quebra de linha ao final |
| `echo -e "mensagem"` | Habilita a interpretação de **caracteres especiais** (ex: `\n` para nova linha, `\t` para tabulação) |

### `touch` — Criar arquivos e manipular timestamps

| Comando | Descrição |
|---|---|
| `touch <arquivo>` | Cria um arquivo vazio, ou **atualiza o timestamp** de um existente |
| `touch -a <arquivo>` | Atualiza apenas o **tempo de acesso** (atime) |
| `touch -m <arquivo>` | Atualiza apenas o **tempo de modificação** (mtime) |
| `touch -c <arquivo>` | Não cria o arquivo caso ele **não exista** — apenas atualiza timestamps de existentes |
| `touch -t [[CC]YY]MMDDhhmm[.ss] <arquivo>` | Define um **timestamp específico** para o arquivo |

### `time` — Medir tempo de execução

| Comando | Descrição |
|---|---|
| `time <comando>` | Executa o comando e exibe ao final o tempo **real**, de **CPU do usuário** e de **CPU do sistema** consumidos |

### `seq` — Sequência de números

| Comando | Descrição |
|---|---|
| `seq N` | Exibe sequência de **1 até N**, passo 1 |
| `seq N M` | Exibe sequência de **N até M**, passo 1 |
| `seq N P M` | Exibe sequência de **N até M** com passo **P** (ex: `seq 1 2 10` → 1 3 5 7 9) |

### `sync` — Gravar buffer do kernel em disco

| Comando | Descrição |
|---|---|
| `sync` | Força a gravação imediata dos dados do **buffer do kernel** no disco — o kernel faz isso automaticamente de tempos em tempos, mas este comando garante a escrita imediata |

---

## Usuários e Permissões

| Comando | Descrição |
|---|---|
| `su` | Alterna para o usuário root — **requer a senha do root** |
| `su -` | Alterna para root carregando o ambiente completo do root |
| `sudo <comando>` | Executa um único comando com privilégios de root usando a **senha do próprio usuário** (sem precisar da senha do root). O usuário precisa estar no grupo `sudo` |
| `sudo su` | Concede um shell root completo usando a **senha do usuário atual** (sem a senha do root) |
| `adduser <usuario> sudo` | Adiciona um usuário ao grupo `sudo`, concedendo privilégios de superusuário |
| `deluser <usuario> sudo` | Remove um usuário do grupo `sudo` |

> **Diferença prática:** `su` exige a senha do root; `sudo` exige apenas a senha do usuário atual, desde que ele esteja autorizado. `sudo su` é uma forma de obter shell root sem conhecer a senha do root.

---

## Interação Direta com o Kernel

> **Atenção:** Os comandos abaixo interagem diretamente com o kernel via `sysrq`. Use apenas em situações extremas, quando o sistema não responde de nenhuma outra forma.

| Comando | Descrição |
|---|---|
| `echo b > /proc/sysrq-trigger` | **Reinicia** a máquina imediatamente, sem desmontar sistemas de arquivos |
| `echo o > /proc/sysrq-trigger` | **Desliga** a máquina imediatamente |