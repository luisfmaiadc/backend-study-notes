# Sistema de Arquivos do Linux

## LSB — Linux Standard Base

O **LSB** é um conjunto de normas e estratégias definido em conjunto por diversas distribuições Linux com o objetivo de padronizar o uso do kernel entre elas.

Distribuições que seguem o LSB devem, entre outras coisas, adotar o **FHS** como padrão de estrutura de diretórios.

---

## FHS — Filesystem Hierarchy Standard

O **FHS** define a estrutura e organização dos diretórios no Linux, garantindo consistência entre as distribuições que o adotam.

### Estrutura de Diretórios

#### Diretórios raiz essenciais

| Diretório | Descrição |
|---|---|
| `/bin` | Binários de usuários essenciais ao boot |
| `/sbin` | Binários do superusuário (root) essenciais ao boot |
| `/boot` | Arquivos de inicialização do sistema — inclui o GRUB e o kernel (`vmlinuz`, ~5MB) |
| `/dev` | Arquivos de dispositivos do sistema |
| `/etc` | Arquivos de configuração globais da máquina |
| `/home` | Diretórios pessoais dos usuários comuns |
| `/root` | Diretório home exclusivo do superusuário (root) |
| `/lib` | Bibliotecas dinâmicas compartilhadas por `/bin` e `/sbin`, e módulos do kernel |
| `/lib32` / `/lib64` | Bibliotecas específicas para arquiteturas 32 ou 64 bits |
| `/lost+found` | Arquivos recuperados após desligamento abrupto, para prevenir corrompimento |
| `/media` | Ponto de montagem para mídias removíveis (pendrive, CD, DVD) |
| `/mnt` | Ponto de montagem temporário (ex: imagens ISO) |
| `/opt` | Software externo instalado fora do sistema de empacotamento padrão |
| `/proc` | Filesystem virtual provido pelo kernel — expõe informações dinâmicas do sistema (CPU, processos, etc.) |
| `/run` | Dados dinâmicos sobre processos em execução desde o último boot (tmpfs) |
| `/srv` | Dados estáticos de serviços de servidor (HTTP, FTP, Git) |
| `/sys` | Informações sobre o sistema: dispositivos, drivers, firmware, módulos e energia |
| `/tmp` | Arquivos temporários — geralmente apagados no reboot |
| `/var` | Dados variáveis gerados pelo sistema em uso (logs, cache, e-mail, spool, etc.) |

#### `/etc` — Subdiretórios relevantes

| Diretório | Descrição |
|---|---|
| `/etc/opt` | Configurações de aplicativos instalados em `/opt` |
| `/etc/X11` | Configurações do X Window System |

#### `/usr` — Segunda hierarquia

`/usr` (*Unix System Resources*) contém arquivos não essenciais ao boot, mas necessários para aplicações do usuário. Segue uma hierarquia similar à raiz:

| Diretório | Descrição |
|---|---|
| `/usr/bin` | Binários de usuários não essenciais ao boot ou recovery |
| `/usr/sbin` | Binários do superusuário não essenciais ao boot |
| `/usr/lib` | Bibliotecas não essenciais ao boot |
| `/usr/include` | Headers padrão para compilação |
| `/usr/share` | Dados compartilhados independentes de arquitetura |
| `/usr/src` | Código-fonte armazenado na máquina |
| `/usr/local` | Binários instalados fora do sistema de empacotamento — hierarquia terciária |
| `/usr/X11R6` | X Window System versão 11R6 |

#### `/var` — Subdiretórios relevantes

| Diretório | Descrição |
|---|---|
| `/var/log` | Arquivos de log do sistema |
| `/var/mail` | Caixas de e-mail dos usuários (formato mailbox) |
| `/var/lock` | Arquivos de lock para controle de recursos em uso |
| `/var/run` | Dados sobre a execução do sistema desde o boot (daemons e usuários) |
| `/var/spool` | Filas de tarefas (impressão, cache de pacotes, proxy) |
| `/var/tmp` | Arquivos temporários em modo multi-usuário |

---

## Tipos de Arquivos

No Linux, tudo é representado como arquivo — incluindo dispositivos e diretórios. O tipo é identificado pelo primeiro caractere na listagem detalhada (`ls -l`):

| Símbolo | Tipo |
|---|---|
| `-` | Arquivo comum |
| `d` | Diretório |
| `l` | Link simbólico |
| `b` | Dispositivo de bloco (armazenamento) |
| `c` | Dispositivo de caractere (transferência de dados) |

### Dispositivos em `/dev`

Os arquivos em `/dev` representam os dispositivos do sistema. Existem dois tipos:

- **Dispositivos de bloco** — armazenam dados em blocos (ex: discos rígidos, SSDs, pendrives)
- **Dispositivos de caractere** — transferem dados caractere a caractere (ex: terminais, portas seriais)

> Discos são identificados como `sda`, `sdb`, etc. Partições primárias como `sda1`, `sda2`; partições lógicas a partir de `sda5`.

---

## PID — Process Identification

Cada processo em execução no Linux recebe um identificador único chamado **PID** (*Process Identification*). É por meio do PID que o sistema gerencia, monitora e encerra processos em curso.