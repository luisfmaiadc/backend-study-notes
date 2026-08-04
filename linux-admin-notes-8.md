# Atributos, Extração e Comparação de Arquivos

## Atributos de Arquivo — `chattr` e `lsattr`

> **Atributos ≠ Permissões.** Permissões controlam quem pode ler, escrever ou executar. Atributos controlam **comportamentos do sistema de arquivos** sobre o arquivo, independente do usuário.

| Comando | Descrição |
|---|---|
| `chattr <atributo> <arquivo>` | Altera os atributos do arquivo ou diretório |
| `lsattr <arquivo>` | Lista os atributos do arquivo |
| `lsattr -d <diretório>` | Lista os atributos do **diretório em si**, não do seu conteúdo |
| `lsattr -R <diretório>` | Lista os atributos de forma **recursiva** |

### Sintaxe do `chattr`

| Operador | Significado |
|---|---|
| `+atributo` | **Adiciona** o atributo ao arquivo |
| `-atributo` | **Remove** o atributo do arquivo |
| `=atributos` | **Define** exatamente os atributos informados (substitui os existentes) |

```bash
chattr +i teste1       # Adiciona imutabilidade ao arquivo
chattr -i teste1       # Remove imutabilidade
chattr =ic teste1      # Define os atributos i e c de uma vez
```

### Atributos disponíveis

| Atributo | Descrição |
|---|---|
| `i` | **Imutável** — impede qualquer alteração, renomeação ou exclusão, inclusive pelo root |
| `a` | **Append-only** — permite apenas adicionar conteúdo ao final do arquivo, sem modificar o que já existe |
| `c` | **Compressão automática** — o kernel compacta o arquivo em disco e descompacta transparentemente ao lê-lo |
| `s` | **Deleção segura** — ao excluir, os blocos do arquivo são zerados em disco (mais lento, mas garante exclusão irrecuperável) |
| `S` | **Sync imediato** — dados em memória são gravados no disco imediatamente, sem esperar o buffer |
| `D` | **Sync de diretório** — mesmo comportamento do `S`, aplicado a diretórios |
| `d` | **Sem backup** — arquivo ou diretório é ignorado pelo utilitário `dump` em backups |

---

## Extração de Conteúdo — `cut`

`cut` extrai partes específicas do conteúdo de um arquivo ou entrada, como colunas ou intervalos de caracteres.

| Comando | Descrição |
|---|---|
| `cut -d ":" -f 1 <arquivo>` | Define `:` como **delimitador** e extrai o **1º campo** de cada linha |
| `cut -d ":" -f 1,3 <arquivo>` | Extrai os campos **1 e 3** |
| `cut -d ":" -f 1-3 <arquivo>` | Extrai os campos de **1 a 3** (intervalo) |
| `cut -b 1-4 <arquivo>` | Extrai os **bytes** de posição 1 a 4 de cada linha |
| `cut -c 1-4 <arquivo>` | Extrai os **caracteres** de posição 1 a 4 — semelhante ao `-b`, mas ignora espaços |

```bash
# Exemplo prático: exibir apenas os nomes de usuários do sistema
cut -d ":" -f 1 /etc/passwd
```

---

## Comparação de Arquivos

### `cmp` — Comparação byte a byte

| Comando | Descrição |
|---|---|
| `cmp <arquivo1> <arquivo2>` | Compara dois arquivos e retorna o **byte e linha** onde a primeira diferença ocorre |
| `cmp -s <arquivo1> <arquivo2>` | Modo **silencioso** — não exibe saída, apenas retorna o código de saída: `0` se idênticos, `1` se diferentes. Útil em scripts |

### `diff` — Comparação linha a linha

Mais legível que o `cmp` — exibe as **linhas** que diferem entre dois arquivos, em vez dos bytes.

| Comando | Descrição |
|---|---|
| `diff <arquivo1> <arquivo2>` | Exibe as linhas diferentes entre os dois arquivos |
| `diff -u <arquivo1> <arquivo2>` | Exibe a diferença em formato **unificado** (unified diff) — mais legível, com contexto das linhas ao redor |
| `diff -r <dir1> <dir2>` | Compara dois diretórios **recursivamente**, incluindo diferenças nos arquivos internos |

> O formato unificado (`-u`) é o padrão para geração de **patches** — arquivos que descrevem apenas as mudanças entre versões.

### `patch` — Aplicar mudanças incrementais

`patch` aplica as diferenças geradas pelo `diff` a um arquivo original, atualizando apenas o que mudou — sem substituir o arquivo inteiro.

| Comando | Descrição |
|---|---|
| `patch <arquivo> <arquivo.patch>` | Aplica o patch ao arquivo informado |
| `patch -p1 <arquivo.patch>` | Remove **1 nível** do caminho de arquivo dentro do patch — necessário quando o patch foi gerado a partir de um diretório pai (ex: `a/arquivo.txt` vira `arquivo.txt`) |
| `patch -N <arquivo.patch>` | Não aplica patches que **já foram aplicados** anteriormente |
| `patch -R <arquivo.patch>` | **Reverte** um patch já aplicado, restaurando o estado anterior |

---

## Localização de Comandos — `whereis`

| Comando | Descrição |
|---|---|
| `whereis <comando>` | Exibe os caminhos do **binário**, do **código-fonte** e da **página de manual** do comando informado |

> Diferente do `which`, que localiza apenas o binário no `$PATH`, o `whereis` busca também a documentação e fontes do comando.