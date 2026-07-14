# Curingas (Wildcards)

Curingas são caracteres especiais usados para representar padrões em nomes de arquivos e diretórios. Funcionam com qualquer comando que aceite caminhos como argumento: `ls`, `rm`, `cp`, `mv`, `find`, etc.

---

## `*` — Zero ou mais caracteres

Substitui nenhum, um ou qualquer sequência de caracteres em qualquer posição.

| Exemplo | Resultado |
|---|---|
| `ls *.txt` | Lista todos os arquivos com extensão `.txt` |
| `ls relatorio*` | Lista tudo que começa com "relatorio" |
| `rm *.log` | Remove todos os arquivos `.log` do diretório atual |
| `ls m*` | Lista todos os arquivos e diretórios que começam com "m" |

---

## `?` — Exatamente um caractere

Substitui **exatamente um** caractere na posição informada.

| Exemplo | Resultado |
|---|---|
| `ls arquivo?` | Corresponde a `arquivo1`, `arquivoA` — mas **não** a `arquivo12` |
| `ls m?o` | Corresponde a `mao`, `mio`, `mvo`, etc. |
| `ls log_?.txt` | Corresponde a `log_1.txt`, `log_a.txt`, mas não a `log_10.txt` |

---

## `[...]` — Classe de caracteres

Substitui **exatamente um** caractere dentro de um conjunto ou intervalo definido.

| Exemplo | Resultado |
|---|---|
| `ls m[a-d]` | Corresponde a `ma`, `mb`, `mc` ou `md` |
| `ls arquivo[123]` | Corresponde a `arquivo1`, `arquivo2` ou `arquivo3` |
| `ls [a-z]*` | Lista tudo que começa com uma letra minúscula |

### Negação com `^`

O `^` dentro dos colchetes **nega** o padrão — seleciona o que **não** corresponde ao conjunto.

| Exemplo | Resultado |
|---|---|
| `ls m[^a-bc]` | Corresponde a "m" seguido de qualquer caractere **exceto** a, b e c |
| `ls [^0-9]*` | Lista arquivos cujo nome **não** começa com número |

---

## `{...}` — Expansão de strings

Expande para múltiplos padrões definidos entre chaves, separados por vírgula. Diferente dos outros curingas, `{}` é uma **expansão do shell** — o Bash resolve as combinações antes de executar o comando.

| Exemplo | Resultado |
|---|---|
| `ls x{zd,ze,zm}*` | Lista arquivos que começam com `xzd`, `xze` ou `xzm` |
| `mkdir {docs,imgs,scripts}` | Cria três diretórios de uma só vez |
| `cp arquivo.{txt,bak} /destino` | Copia `arquivo.txt` e `arquivo.bak` juntos |
| `touch log_{01,02,03}.txt` | Cria `log_01.txt`, `log_02.txt` e `log_03.txt` |