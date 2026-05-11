# ⌨️ Guia de Comandos do VS Code

> Referência rápida de atalhos e comandos úteis para o dia a dia no VS Code.

---

## 🗂️ Navegação e Interface

| Atalho | Ação | Descrição |
|---|---|---|
| `Ctrl + B` | Abrir/Fechar Barra Lateral | Exibe ou oculta o painel lateral (navegador de arquivos, extensões etc.). Útil para ganhar espaço de tela ao focar na edição. |
| `Ctrl + Shift + E` | Explorador de Arquivos | Foca diretamente no painel do Explorer, permitindo navegar pela estrutura de pastas do projeto sem usar o mouse. |
| `Ctrl + Shift + X` | Extensões | Abre o painel de extensões para instalar, desabilitar ou gerenciar plugins (ex: Extension Pack for Java). |
| `Ctrl + \` | Dividir Editor | Divide a tela do editor em duas colunas, permitindo visualizar dois arquivos simultaneamente. |
| `Ctrl + 1 / 2 / 3` | Alternar entre Grupos de Editor | Move o foco para o primeiro, segundo ou terceiro grupo de abas abertas no editor dividido. |
| `Ctrl + Tab` | Alternar entre Abas | Navega ciclicamente pelas abas abertas no editor atual. |

---

## 📁 Gerenciamento de Arquivos

| Atalho | Ação | Descrição |
|---|---|---|
| `Ctrl + N` | Novo Arquivo | Cria um novo arquivo sem título. Para definir a linguagem, use `Ctrl + Shift + P` → *Change Language Mode*. |
| `Ctrl + P` | Abrir Arquivo | Abre a busca rápida de arquivos do projeto. Digite parte do nome para localizar e abrir o arquivo desejado. Adicione `:` seguido de um número para ir direto a uma linha (ex: `Main.java:42`). |
| `Ctrl + S` | Salvar | Salva o arquivo atual. |
| `Ctrl + Shift + S` | Salvar Como | Salva uma cópia do arquivo com um novo nome ou localização. |
| `Ctrl + W` | Fechar Aba | Fecha a aba atualmente focada. |
| `Ctrl + Shift + T` | Reabrir Aba Fechada | Reabre a última aba que foi fechada. |

---

## 🔍 Busca e Navegação no Código

| Atalho | Ação | Descrição |
|---|---|---|
| `Ctrl + Shift + P` | Paleta de Comandos | Acessa todos os comandos disponíveis do VS Code e extensões instaladas. É o "hub" principal — use para formatar código, alterar tema, instalar extensões e muito mais. |
| `Ctrl + Shift + O` | Navegar por Símbolos no Arquivo | Lista todos os símbolos do arquivo atual (métodos, classes, atributos). Permite navegar diretamente para um método específico sem rolar o código. Muito útil em classes grandes. |
| `Ctrl + T` | Buscar Símbolo no Workspace | Semelhante ao anterior, mas busca em todos os arquivos do projeto — ideal para localizar uma classe ou interface pelo nome. |
| `Ctrl + G` | Ir para Linha | Salta o cursor diretamente para uma linha específica pelo número. |
| `F12` | Ir para Definição | Navega até onde uma classe, método ou variável foi definido. |
| `Alt + F12` | Espreitar Definição | Exibe a definição em um pop-up inline sem sair do arquivo atual. |
| `Shift + F12` | Encontrar Referências | Lista todos os locais do projeto onde aquele símbolo é utilizado. Essencial para entender o impacto de uma alteração antes de refatorar. |
| `Ctrl + Shift + F12` | Encontrar Implementações | Lista todas as implementações concretas de uma interface ou método abstrato. Muito útil em projetos com uso intenso de polimorfismo. |
| `Ctrl + F` | Buscar no Arquivo | Abre a barra de busca para encontrar texto no arquivo atual. |
| `Ctrl + Shift + F` | Buscar no Workspace | Busca um texto em todos os arquivos do projeto. Suporta expressões regulares. |
| `Alt + ← / →` | Navegar no Histórico | Volta ou avança no histórico de posições de cursor (similar ao botão "voltar" de um browser). Ideal após usar `F12` para retornar ao ponto de origem. |

---

## ✏️ Edição de Código

| Atalho | Ação | Descrição |
|---|---|---|
| `Shift + Alt + ↓` | Duplicar Linha | Copia a linha atual e a insere imediatamente abaixo. Útil para criar variações de uma instrução sem redigitá-la. |
| `Alt + ↑ / ↓` | Mover Linha | Move a linha atual (ou o bloco selecionado) para cima ou para baixo sem precisar recortar e colar. |
| `Ctrl + Shift + K` | Deletar Linha | Remove a linha inteira onde o cursor está posicionado, sem precisar selecionar o conteúdo. |
| `Ctrl + /` | Comentar/Descomentar Linha | Alterna o comentário da linha ou seleção atual. Em Java, usa `//`. |
| `Ctrl + Shift + A` | Comentário em Bloco | Envolve a seleção com `/* ... */`. |
| `Tab / Shift + Tab` | Indentar / Desindentar | Adiciona ou remove um nível de indentação na linha ou seleção atual. |
| `Ctrl + Z` | Desfazer | Reverte a última ação realizada. |
| `Ctrl + Shift + Z` | Refazer | Reaplica a última ação desfeita. |
| `Ctrl + D` | Selecionar Próxima Ocorrência | Seleciona a próxima ocorrência da palavra atualmente selecionada. Permite editar múltiplas ocorrências ao mesmo tempo (ver seção MultiCursor). |

---

## 🖱️ Edição MultiCursor

O MultiCursor permite posicionar vários cursores ao mesmo tempo, editando múltiplos pontos do código simultaneamente.

### Formas de ativar

| Atalho | Como funciona |
|---|---|
| `Alt + Click` | Adiciona um cursor no ponto clicado. Cada clique adiciona um novo cursor independente. |
| `Ctrl + Alt + ↑ / ↓` | Adiciona cursores na linha acima ou abaixo do cursor atual, na mesma coluna. |
| `Ctrl + D` | Seleciona a palavra sob o cursor e, a cada novo `Ctrl + D`, seleciona a próxima ocorrência igual — permitindo editar todas ao mesmo tempo. |
| `Ctrl + Shift + L` | Seleciona **todas** as ocorrências da palavra/seleção atual no arquivo de uma vez. |

### Exemplos de uso

**Exemplo 1 — Renomear variável local em múltiplos pontos com `Ctrl + D`:**
```java
// Antes: clique sobre "valor" e pressione Ctrl + D repetidamente
int valor = 10;
System.out.println(valor);
return valor;

// Após selecionar todas as ocorrências de "valor", digite "total":
int total = 10;
System.out.println(total);
return total;
```

**Exemplo 2 — Adicionar prefixo em várias linhas com `Ctrl + Alt + ↓`:**
```java
// Posicione o cursor no início de "String" e pressione Ctrl + Alt + ↓ para
// adicionar cursores nas linhas abaixo, depois digite "private " em todas:

// Antes:
String nome;
String email;
String telefone;

// Depois:
private String nome;
private String email;
private String telefone;
```

**Exemplo 3 — Adicionar aspas em múltiplos valores com `Alt + Click`:**
```java
// Use Alt + Click para clicar antes de cada valor e depois no fim de cada um:

// Antes:
String[] frutas = {maçã, banana, uva};

// Depois:
String[] frutas = {"maçã", "banana", "uva"};
```

---

## 🛠️ Refatoração e Produtividade

| Atalho | Ação | Descrição |
|---|---|---|
| `F2` | Renomear Símbolo | Renomeia uma variável, método ou classe em **todos os arquivos do projeto** de forma segura, respeitando o escopo. Diferente de um "find & replace", o VS Code entende o contexto do código. |
| `Ctrl + .` | Ações Rápidas e Refatorações | Abre o menu de sugestões contextuais. Permite extrair um trecho de código para um novo método, gerar getters/setters, implementar métodos de interfaces, corrigir imports ausentes e muito mais. É o equivalente ao `Alt + Enter` do IntelliJ. |
| `Ctrl + Shift + R` | Refatorar (menu completo) | Exibe todas as opções de refatoração disponíveis para o símbolo selecionado (renomear, mover, extrair etc.). |
| `Shift + Alt + F` | Formatar Documento | Formata o arquivo inteiro de acordo com as regras de estilo configuradas (requer formatador instalado, como o do Extension Pack for Java). |
| `Ctrl + K Ctrl + F` | Formatar Seleção | Formata apenas o trecho de código selecionado. |

---

## ▶️ Execução e Debug

| Atalho | Ação | Descrição |
|---|---|---|
| `F5` | Executar com Debug | Inicia a aplicação em modo de depuração, respeitando os breakpoints definidos. Requer um arquivo `launch.json` configurado (gerado automaticamente pelo Extension Pack for Java na primeira execução). |
| `Ctrl + F5` | Executar sem Debug | Executa a aplicação sem iniciar o depurador — mais rápido para testes rápidos. |
| `F9` | Adicionar/Remover Breakpoint | Marca ou desmarca a linha atual como ponto de parada durante o debug. |
| `F10` | Step Over | Avança para a próxima linha durante o debug, sem entrar em chamadas de método. |
| `F11` | Step Into | Entra dentro da chamada de método na linha atual para inspecionar sua execução. |
| `Shift + F11` | Step Out | Sai do método atual e retorna ao chamador. |
| `Ctrl + Shift + `` ` | Abrir Terminal Integrado | Abre o terminal diretamente dentro do VS Code. Permite executar comandos Maven (`mvn`), Gradle ou qualquer comando de shell sem sair do editor. |

---

## 💡 Dicas para Desenvolvimento Java

- Instale o **Extension Pack for Java** (Microsoft) — ele inclui suporte a linguagem, debug, Maven, Gradle, testes e muito mais.
- Use `Ctrl + Shift + P` → *Java: Clean Java Language Server Workspace* quando o IntelliSense apresentar comportamento inesperado.
- Use `Ctrl + Shift + P` → *Java: Organize Imports* para organizar e remover imports não utilizados.
- O atalho `Ctrl + .` sobre uma interface ou classe abstrata oferece a opção *Implement all abstract methods* automaticamente.