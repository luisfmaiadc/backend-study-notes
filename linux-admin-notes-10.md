# Multiplexadores de Terminal

Multiplexadores permitem utilizar **vários terminais dentro de uma única janela**, além de manter sessões em execução em segundo plano — mesmo após fechar o terminal ou encerrar a conexão SSH.

---

## screen

`screen` cria sessões com múltiplas janelas (telas) dentro do mesmo terminal.

```bash
screen              # Inicia uma nova sessão screen
screen -x           # Reconecta a uma sessão que estava desconectada (mas seguia em execução)
screen -r           # Reconecta a uma sessão detachada específica
screen -ls          # Lista todas as sessões screen ativas
```

### Atalhos internos (`Ctrl+A` + tecla)

> Todos os atalhos do `screen` são acionados com o prefixo **`Ctrl+A`**, seguido da tecla indicada.

| Atalho | Ação |
|---|---|
| `Ctrl+A` → `C` | **Cria** uma nova tela (janela) |
| `Ctrl+A` → `W` | **Lista** as telas abertas na sessão |
| `Ctrl+A` → `N` | **Navega** para a tela de número `N` |
| `Ctrl+A` → `A` | **Alterna** entre as duas últimas telas visitadas |
| `Ctrl+A` → `D` | **Desconecta** (detach) a sessão — ela segue em execução em segundo plano sem interrupção |

---

## tmux

`tmux` oferece a mesma proposta do `screen`, com mais recursos: divisão de painéis, sessões nomeadas e configuração mais flexível.

```bash
tmux                # Inicia uma nova sessão
tmux new            # Cria uma nova janela
tmux ls             # Lista as sessões tmux ativas
tmux attach         # Reconecta à última sessão ativa
```

### Atalhos internos (`Ctrl+B` + tecla)

> Todos os atalhos do `tmux` são acionados com o prefixo **`Ctrl+B`**, seguido da tecla indicada.

#### Janelas

| Atalho | Ação |
|---|---|
| `Ctrl+B` → `C` | **Cria** uma nova janela |
| `Ctrl+B` → `N` | **Navega** para a janela de número `N` |
| `Ctrl+B` → `0` | Vai diretamente para a **janela 0** |
| `Ctrl+B` → `1` | Vai diretamente para a **janela 1** |
| `Ctrl+B` → `D` | **Desconecta** (detach) a sessão — continua em execução em segundo plano |

#### Divisão de painéis

| Atalho | Ação |
|---|---|
| `Ctrl+B` → `Shift+"` | Divide a tela em **dois painéis horizontais** |
| `Ctrl+B` → `%` | Divide a tela em **dois painéis verticais** |

#### Navegação entre painéis

| Atalho | Ação |
|---|---|
| `Ctrl+B` → `↑` | Move para o painel **acima** |
| `Ctrl+B` → `↓` | Move para o painel **abaixo** |
| `Ctrl+B` → `←` | Move para o painel à **esquerda** |
| `Ctrl+B` → `→` | Move para o painel à **direita** |

---

## screen vs tmux

| Característica | screen | tmux |
|---|---|---|
| Divisão de painéis | Limitada | Nativa e flexível |
| Sessões nomeadas | Não | Sim |
| Configuração | Simples | Mais completa |
| Disponibilidade | Presente na maioria das distros | Pode precisar de instalação |
| Prefixo de atalhos | `Ctrl+A` | `Ctrl+B` |