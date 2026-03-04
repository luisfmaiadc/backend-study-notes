# 📌 Spring Batch – Base Conceitual e Arquitetura

## 1. O que é processamento Batch?

O termo Batch significa literalmente *lote*.
Um sistema batch é um sistema que:
- Processa uma quantidade finita de dados
- Executa sem interação humana
- Normalmente roda de forma agendada
- Pode processar grandes volumes de dados

### 📚 Contexto Histórico

Historicamente, o processamento batch foi dominado por:
- COBOL
- Executado em *Mainframe*

Mainframes são computadores de grande porte, com altíssima capacidade de processamento e armazenamento, porém com alto custo.

## 2. O que é o Spring Batch?

Spring Batch é um framework do ecossistema Spring para processamento batch em aplicações Java.

Ele oferece:
- Controle de execução
- Controle transacional
- Restart automático
- Gerenciamento de metadados
- Tratamento de falhas
- Escalabilidade

## 3. Banco de Dados é Obrigatório?

Sim.

O Spring Batch exige um banco de dados configurado, pois ele precisa armazenar:
- Metadados de execução
- Estado dos jobs
- Controle de restart
- Estatísticas

Esses dados ficam em tabelas como:
- BATCH_JOB_INSTANCE
- BATCH_JOB_EXECUTION
- BATCH_STEP_EXECUTION
- BATCH_JOB_EXECUTION_PARAMS

Esse conjunto é chamado de **JobRepository** metadata schema.

## 4. Arquitetura Central do Spring Batch

### Job

Um Job representa um trabalho batch completo.

Ele pode ser entendido como:
- Uma máquina de estados composta por Steps e transições.

Um Job:
- Possui um nome
- Possui parâmetros
- Contém um ou mais Steps
- Pode ter fluxo condicional

### Step

Um Step é uma etapa dentro do Job.

Cada Step:
- Tem sua própria transação
- Possui estado próprio
- Pode falhar independentemente

Um Job pode conter múltiplos Steps encadeados.

## 5. Componentes Principais

### 1️⃣ JobLauncher

Responsável por:
- Iniciar o Job
- Validar parâmetros
- Reexecutar quando permitido

É a porta de entrada do processamento.

### 2️⃣ JobRepository

Responsável por:
- Persistir metadados
- Controlar estado
- Permitir restart

Sem ele o Spring Batch não funciona corretamente.

## 6. JobInstance vs JobExecution (Ponto Muito Importante)

Aqui está um ponto que geralmente gera confusão.

### JobInstance (Execução Lógica)

Representa:

Uma execução lógica do Job baseada nos parâmetros.

Regra:

```
Job + Parâmetros = 1 JobInstance
```

Se os parâmetros mudarem → nova JobInstance.

### JobExecution (Execução Física)

Representa:

Uma tentativa de execução daquela JobInstance.

Exemplo:
- JobInstance #1
    - JobExecution #1 → FAILED
    - JobExecution #2 → FAILED
    - JobExecution #3 → COMPLETED

Um Job pode ter múltiplas JobInstances.
Cada JobInstance pode ter múltiplas JobExecutions.

## 7. Parâmetros do Job

Os parâmetros:
- Determinam a identidade do JobInstance
- São imutáveis
- Influenciam a lógica

Se você tentar executar um Job com os mesmos parâmetros de uma execução COMPLETED, o Spring lança:

```java
JobInstanceAlreadyCompleteException
```

### RunIdIncrementer()

Serve para:
- Adicionar automaticamente um parâmetro incremental
- Permitir múltiplas execuções

#### ⚠️ Cuidado:

Ao usar incrementador:
- Você perde a capacidade de restart daquela execução anterior
- Pois cada execução vira uma nova JobInstance

Use apenas quando quiser múltiplas execuções independentes.

## 8. Injetando Parâmetros

Para acessar parâmetros:

```java
@Value("#{jobParameters['nomeDoParametro']}")
```

Requisitos:
- O bean precisa estar anotado com `@StepScope`
- Precisa ser um bean Spring

Isso garante que o valor seja resolvido no momento da execução do Step.

## 9. Tipos de Step

Existem dois principais:

### 9.1 Tasklet

Ideal para:
- Pequenas tarefas
- Execuções simples
- Comandos únicos

Exemplos:
- Limpeza de arquivos
- Criação de diretórios
- Envio simples de e-mail

Ela executa até retornar:

```java
RepeatStatus.FINISHED
```

### 9.2 Chunk-Oriented Processing

Utilizado para grandes volumes de dados.

Fluxo:

```
Read → Process → Write
```

**Cada chunk**:
- Possui sua própria transação
- É commitado ao final da escrita

#### Commit Interval

Define:
    Quantos itens serão processados antes do commit.

Exemplo:

```java
.<Integer, String>chunk(100)
```

- Lê 100
- Processa 100
- Escreve 100
- Commit

Se falhar antes do commit → rollback daquele chunk.

## 10. Componentes do Chunk

### ItemReader

Interface:

```java
ItemReader<T>
```

Método:

```java
T read()
```

- Retorna item
- Retorna null quando termina

Enquanto não retornar null, o framework continua lendo.

**Fontes comuns:**
- Arquivos (Flat)
- XML
- JSON
- Banco de Dados
- Filas
- APIs

**🔁 Restart e Checkpoint**

O Spring Batch salva:
- Último item processado
- Estado do StepExecution

Isso permite reiniciar de onde parou (dependendo da implementação do Reader).

#### FlatFileItemReader

Possui:

**LineTokenizer**: Divide a linha em tokens.

**FieldSetMapper**: Converte tokens em objeto de domínio.

Implementação pessoal: [sbatch-pending-email-sender](https://github.com/luisfmaiadc/sbatch-pending-email-sender)

#### MultiResourceItemReader

Permite ler múltiplos arquivos como se fosse um só fluxo lógico.

Implementação base: [Aula - Udemy](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19268374#overview)

## 11. Leitura com JDBC

O Spring Batch oferece:

### Cursor
- Executa 1 consulta
- Mantém cursor aberto
- Mais rápido
- Mais consumo de memória
- Pode segurar conexão por muito tempo
- Implementação pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

### Paginação
- Executa múltiplas consultas
- Carrega páginas limitadas
- Menor consumo de memória
- Pode ser mais lento
- Implementação base: [Aula - Udemy](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19627774#overview)

Escolha depende de:
- Volume de dados
- Infraestrutura
- Tamanho da memória disponível

## 12. Fault Tolerance (Tolerância a Falhas)

Configuração:

```java
.faultTolerant()
.skip(Exception.class)
.skipLimit(10)
```

### 🔎 O que acontece internamente?

Quando ocorre uma exceção configurada como skip:
- O item é descartado
- O chunk continua
- O contador de skip é incrementado

Se ultrapassar skipLimit → Step falha.

O Spring **não tenta reprocessar automaticamente** itens que foram skipados.

### Retry

Diferente de Skip:
- Retry tenta reexecutar o item antes de descartá-lo
- Pode ser configurado com:
    - `.retry()`
    - `.retryLimit()`

## 13. Processor

Interface:

```java
ItemProcessor<I, O>
```

Recebe 1 item por vez.

Se retornar:
- Objeto → vai para escrita
- null → item é filtrado

### Tipos Importantes

#### ValidatingItemProcessor
Valida regras de negócio.

Implementação pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

#### CompositeItemProcessor
Encadeia múltiplos processadores.

Implementação pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

#### ClassifierCompositeItemProcessor
Escolhe processador baseado no tipo do item.

Implementação base: [Aula - Udemy](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/18557734#overview)

## 14. Writer

Interface:

```java
ItemWriter<T>
```
Recebe:

```java
List<T>
```

A escrita encerra o chunk.

#### FlatFileItemWriter
- CSV
- Largura fixa

#### JdbcBatchItemWriter
- Executa batch de INSERT/UPDATE
- Alta performance
- Executa em lote dentro da transação do chunk
- Implementação pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

#### MultiResourceItemWriter
- Divide saída em múltiplos arquivos.
- Implementação base: [Aula - Udemy](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19975510#overview)

#### CompositeItemWriter
- Encadeia múltiplos writers.
- Implementação base: [Aula - Udemy](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/20017840?start=330#overview)

#### ClassifierItemWriter
- Implementação pessoal: [sbatch-ecommerce-catalog-aggregator](https://github.com/luisfmaiadc/sbatch-ecommerce-catalog-aggregator)

## 15. Padrão Delegate

O Spring Batch utiliza fortemente o padrão:
    Delegar responsabilidade a outro componente especializado.

Exemplo:
- MultiResource delega ao writer interno
- Composite delega aos writers da lista

## 16. Conexão com múltiplos bancos

Em configurações com múltiplos DataSources:

Declarar corretamente `jdbcUrl`

Separar:
- DataSource do negócio
- DataSource do JobRepository (opcional, mas recomendado)

## 17. Complemento – Generics no Chunk (Ponto Fundamental)

Uma das partes mais importantes do processamento orientado a chunk é entender como os Generics fluem entre Reader, Processor e Writer. Quando declaramos:

```java
.<I, O>chunk(100)
```

Estamos definindo:
- *I (Input)* → Tipo que será LIDO pelo ItemReader
- *O (Output)* → Tipo que será PRODUZIDO pelo `ItemProcessor` e ESCRITO pelo `ItemWriter`

### 🔁 Fluxo de Tipos no Chunk

Se declararmos:

```java
.<Cliente, ClienteDTO>chunk(100)
```

O fluxo será:

```
ItemReader<Cliente>
        ↓
ItemProcessor<Cliente, ClienteDTO>
        ↓
ItemWriter<ClienteDTO>
```

### 📌 Como cada interface utiliza Generics

#### - ItemReader<T>

```java
ItemReader<T>
```

- T → Tipo que será retornado pelo método `read()`
- É o tipo bruto vindo da fonte de dados

#### - ItemProcessor<I, O>

```java
ItemProcessor<I, O>
```

- I → Tipo recebido do Reader
- O → Tipo devolvido para o Writer

⚠️ Se retornar null, o item é filtrado e não segue para o Writer.


#### - ItemWriter<T>

```java
ItemWriter<T>
```

Recebe:

```java
void write(List<? extends T> items)
```

- T → Tipo que veio do Processor (ou Reader, se não houver Processor)

### 🔄 E se não houver Processor?

Se você configurar:

```java
.<Cliente, Cliente>chunk(100)
```

Significa:
- O Reader retorna Cliente
- Não há transformação
- O Writer recebe Cliente

## 18. ExecutionContext (Fundamental para Restart)

O ExecutionContext é um dos conceitos mais importantes do Spring Batch.

Ele é:
- Um mapa chave-valor persistido no banco de dados para armazenar estado durante a execução.

Internamente funciona como:

```java
Map<String, Object>
```

Ele é salvo automaticamente pelo framework.

### Para que ele serve?

- Armazenar progresso
- Controlar reinício
- Compartilhar dados entre Steps
- Persistir estado customizado

### Tipos de ExecutionContext

**1️⃣ StepExecutionContext**
- Escopo do Step
- Persistido ao final de cada commit
- Ideal para controle de progresso

**2️⃣ JobExecutionContext**
- Escopo do Job inteiro
- Compartilhado entre Steps

### 🔁 Relação com Restart

Quando um Step falha:
- O Spring Batch lê o ExecutionContext salvo
- Restaura o estado
- Continua do último checkpoint

Isso só funciona corretamente se:
- O Reader for restartable
- O estado estiver sendo salvo corretamente

### Exemplo de uso manual

```java
executionContext.put("ultimaPaginaProcessada", 5);
```

Recuperando:

```java
executionContext.getInt("ultimaPaginaProcessada");
```

### 📌 Importante

O ExecutionContext:
- Só aceita tipos serializáveis
- É persistido como BLOB no banco
- É salvo automaticamente no commit do chunk

## 18. Listeners (Interceptadores de Ciclo de Vida)

Listeners permitem interceptar eventos do ciclo de vida do Job ou Step. São extremamente úteis para:
- Logs
- Auditoria
- Métricas
- Notificações
- Ajustes dinâmicos

### Principais Listeners

#### 🎯 JobExecutionListener

Intercepta:
- Antes do Job iniciar
- Depois do Job finalizar

```java
beforeJob(JobExecution jobExecution)
afterJob(JobExecution jobExecution)
```

Uso comum:
- Enviar e-mail ao finalizar
- Log de início/fim

#### 🎯 StepExecutionListener

Intercepta:
- Antes do Step iniciar
- Depois do Step finalizar

```java
beforeStep(StepExecution stepExecution)
afterStep(StepExecution stepExecution)
```

Pode inclusive alterar o `ExitStatus`

Implementação pessoal: [sbatch-ecommerce-catalog-aggregator](https://github.com/luisfmaiadc/sbatch-ecommerce-catalog-aggregator)

#### 🎯 ChunkListener

Intercepta:
- Antes do chunk
- Depois do chunk
- Em caso de erro

Muito útil para métricas de performance.

#### 🎯 ItemReadListener

Intercepta:
- Antes da leitura
- Depois da leitura
- Erro na leitura

#### 🎯 ItemProcessListener

Intercepta:
- Antes do processamento
- Depois do processamento
- Erro no processamento

#### 🎯 ItemWriteListener

Intercepta:
- Antes da escrita
- Depois da escrita
- Erro na escrita

### 📌 Exemplo de Uso

```java
.stepBuilderFactory.get("step")
    .listener(meuListener)
```

## 19. 🎯 Quando usar ExecutionContext vs Listener?

| Situação                       | Usar                |
| ------------------------------ | ------------------- |
| Guardar estado para restart    | ExecutionContext    |
| Logar eventos                  | Listener            |
| Compartilhar dados entre steps | JobExecutionContext |
| Medir tempo de execução        | Listener            |
| Salvar progresso customizado   | ExecutionContext    |