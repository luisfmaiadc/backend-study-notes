# Spring Batch — Base Conceitual e Arquitetura

---

# 1. Processamento Batch

## Contexto

O termo *batch* significa literalmente **lote**. Um sistema batch é um sistema que:
- Processa uma quantidade finita de dados
- Executa sem interação humana
- Normalmente roda de forma agendada
- Pode processar grandes volumes de dados

Historicamente, o processamento batch foi dominado por COBOL, executado em **Mainframes** — computadores de grande porte com altíssima capacidade de processamento e armazenamento, porém com alto custo.

## O que é o Spring Batch

Spring Batch é um framework do ecossistema Spring para processamento batch em aplicações Java. Ele oferece:
- Controle de execução e controle transacional
- Restart automático e gerenciamento de metadados
- Tratamento de falhas
- Escalabilidade

## Requisitos de Infraestrutura: Banco de Dados

O Spring Batch exige um banco de dados configurado. Ele precisa armazenar metadados de execução, estado dos jobs, controle de restart e estatísticas. Esses dados ficam em tabelas como:

| Tabela | Função |
| --- | --- |
| `BATCH_JOB_INSTANCE` | Registra instâncias lógicas do Job |
| `BATCH_JOB_EXECUTION` | Registra tentativas de execução |
| `BATCH_STEP_EXECUTION` | Registra execuções por Step |
| `BATCH_JOB_EXECUTION_PARAMS` | Armazena os parâmetros de cada execução |

Esse conjunto é chamado de **JobRepository metadata schema**.

---

# 2. Arquitetura Central

## Job

Um Job representa um trabalho batch completo. Pode ser entendido como uma **máquina de estados** composta por Steps e transições.

Um Job:
- Possui um nome e parâmetros
- Contém um ou mais Steps
- Pode ter fluxo condicional entre Steps

## Step

Um Step é uma etapa dentro do Job. Cada Step:
- Tem sua própria transação
- Possui estado próprio no JobRepository
- Pode falhar independentemente dos demais

Um Job pode conter múltiplos Steps encadeados.

## JobLauncher

O `JobLauncher` é a porta de entrada do processamento. É responsável por:
- Iniciar o Job
- Validar parâmetros
- Reexecutar quando permitido

## JobRepository

O `JobRepository` é o componente responsável pela persistência. Sem ele, o Spring Batch não funciona corretamente. É responsável por:
- Persistir metadados de execução
- Controlar o estado de Jobs e Steps
- Permitir restart

---

# 3. Jobs, Instâncias e Parâmetros

## JobInstance vs JobExecution

Esta distinção é um dos pontos que mais gera confusão no Spring Batch.

### JobInstance — Execução Lógica

Representa uma execução lógica do Job com base nos seus parâmetros.

```
Job + Parâmetros = 1 JobInstance
```

Se os parâmetros mudarem, uma nova `JobInstance` é criada.

### JobExecution — Execução Física

Representa uma **tentativa de execução** de uma `JobInstance`.

```
JobInstance #1
    ├── JobExecution #1 → FAILED
    ├── JobExecution #2 → FAILED
    └── JobExecution #3 → COMPLETED
```

Um Job pode ter múltiplas `JobInstance`s. Cada `JobInstance` pode ter múltiplas `JobExecution`s.

## Parâmetros do Job

Os parâmetros:
- Determinam a identidade da `JobInstance`
- São imutáveis
- Podem influenciar a lógica do Job

Tentar executar um Job com os mesmos parâmetros de uma execução `COMPLETED` lança:

```java
JobInstanceAlreadyCompleteException
```

### RunIdIncrementer

O `RunIdIncrementer` adiciona automaticamente um parâmetro incremental a cada execução, permitindo que o Job seja executado múltiplas vezes sem conflito de identidade.

> **Atenção:** ao usar o incrementador, cada execução se torna uma nova `JobInstance`, o que **elimina a capacidade de restart** da execução anterior. Use apenas quando quiser múltiplas execuções independentes.

## Injetando Parâmetros no Step

Para acessar parâmetros dentro de um componente do Step:

```java
@Value("#{jobParameters['nomeDoParametro']}")
```

Requisitos:
- O bean deve estar anotado com `@StepScope`
- Deve ser um bean gerenciado pelo Spring

Isso garante que o valor seja resolvido no momento da execução do Step, e não na inicialização do contexto.

---

# 4. Tipos de Step

## Tasklet

Indicado para tarefas simples e pontuais, como:
- Limpeza de arquivos temporários
- Criação de diretórios
- Envio simples de e-mail

O Tasklet executa e repete até retornar:

```java
RepeatStatus.FINISHED
```

## Chunk-Oriented Processing

Utilizado para processamento de **grandes volumes de dados**. O fluxo básico é:

```
Read → Process → Write
```

Cada chunk possui sua própria transação e é commitado ao final da escrita.

### Commit Interval

Define quantos itens serão lidos, processados e escritos antes de cada commit.

```java
.<Integer, String>chunk(100)
```

Neste exemplo:
- Lê 100 itens
- Processa 100 itens
- Escreve 100 itens
- Commit

Se ocorrer falha antes do commit → rollback do chunk inteiro.

---

# 5. Generics no Chunk

## Fluxo de Tipos

Ao declarar um Step orientado a chunk, os generics definem o **contrato de tipos** entre os três componentes.

```java
.<I, O>chunk(100)
```

- `I (Input)` → tipo lido pelo `ItemReader`
- `O (Output)` → tipo produzido pelo `ItemProcessor` e escrito pelo `ItemWriter`

Exemplo com tipos concretos:

```java
.<Cliente, ClienteDTO>chunk(100)
```

```
ItemReader<Cliente>
        ↓
ItemProcessor<Cliente, ClienteDTO>
        ↓
ItemWriter<ClienteDTO>
```

## Contratos das Interfaces

### ItemReader\<T\>

```java
ItemReader<T>
```

- `T` → tipo retornado pelo método `read()`
- Representa o dado bruto vindo da fonte

### ItemProcessor\<I, O\>

```java
ItemProcessor<I, O>
```

- `I` → tipo recebido do Reader
- `O` → tipo devolvido para o Writer

> Se o método retornar `null`, o item é filtrado e não segue para o Writer.

### ItemWriter\<T\>

```java
ItemWriter<T>
```

Recebe uma lista de itens:

```java
void write(List<? extends T> items)
```

- `T` → tipo vindo do Processor (ou diretamente do Reader, se não houver Processor)

## Chunk sem Processor

Quando não há transformação de tipo, Reader e Writer trabalham com o mesmo tipo:

```java
.<Cliente, Cliente>chunk(100)
```

O Reader retorna `Cliente`, nenhuma transformação ocorre e o Writer recebe `Cliente` diretamente.

---

# 6. ItemReader

## Contrato da Interface

```java
ItemReader<T>
```

Método principal:

```java
T read()
```

- Retorna um item por chamada
- Retorna `null` quando a fonte está esgotada

Enquanto não retornar `null`, o framework continua chamando `read()`.

Fontes comuns suportadas: arquivos flat, XML, JSON, banco de dados, filas e APIs.

## Restart e Checkpoint

O Spring Batch salva o estado do `StepExecution` a cada commit, incluindo o progresso do Reader. Isso permite **reiniciar o Job de onde parou**, dependendo da implementação do Reader utilizada.

## Implementações para Arquivo

### FlatFileItemReader

Lê arquivos de texto linha a linha, utilizando dois componentes:

- `LineTokenizer` — divide a linha em tokens
- `FieldSetMapper` — converte os tokens em objeto de domínio

Projeto pessoal: [sbatch-pending-email-sender](https://github.com/luisfmaiadc/sbatch-pending-email-sender)

### MultiResourceItemReader

Permite ler **múltiplos arquivos** como se fossem um único fluxo lógico.

Referência de aula: [Udemy — MultiResourceItemReader](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19268374#overview)

## Implementações para JDBC

O Spring Batch oferece duas estratégias distintas para leitura de banco de dados:

### JdbcCursorItemReader

| Característica | Detalhe |
| --- | --- |
| Consultas | 1 única consulta |
| Cursor | Mantido aberto durante todo o Step |
| Performance | Mais rápido |
| Memória | Maior consumo |
| Risco | Pode manter a conexão por muito tempo |

Projeto pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

### JdbcPagingItemReader

| Característica | Detalhe |
| --- | --- |
| Consultas | Múltiplas (uma por página) |
| Memória | Menor consumo |
| Performance | Pode ser mais lento |
| Indicado para | Grandes volumes com memória limitada |

Referência de aula: [Udemy — JdbcPagingItemReader](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19627774#overview)

A escolha entre Cursor e Paginação depende de: volume de dados, memória disponível e tempo máximo tolerável de conexão aberta.

---

# 7. ItemProcessor

## Contrato da Interface

```java
ItemProcessor<I, O>
```

Recebe **1 item por vez** e retorna:
- Um objeto → o item segue para a escrita
- `null` → o item é filtrado e descartado

## Implementações Disponíveis

### ValidatingItemProcessor

Aplica validações de regra de negócio ao item. Itens inválidos podem ser descartados ou lançar exceção, conforme configuração.

Projeto pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

### CompositeItemProcessor

Encadeia múltiplos processadores em sequência. A saída de um é a entrada do próximo.

Projeto pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

### ClassifierCompositeItemProcessor

Seleciona dinamicamente qual processador aplicar com base no tipo ou característica do item.

Referência de aula: [Udemy — ClassifierCompositeItemProcessor](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/18557734#overview)

---

# 8. ItemWriter

## Contrato da Interface

```java
ItemWriter<T>
```

Recebe uma lista de itens processados no chunk:

```java
void write(List<? extends T> items)
```

A escrita marca o final do chunk e antecede o commit transacional.

## Implementações Disponíveis

### FlatFileItemWriter

Grava saída em arquivos de texto. Suporta:
- Formato CSV
- Largura fixa

### JdbcBatchItemWriter

Executa operações de INSERT/UPDATE em lote dentro da transação do chunk. Alta performance para escrita em banco de dados.

Projeto pessoal: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

### MultiResourceItemWriter

Divide a saída em múltiplos arquivos conforme um critério configurável (ex.: limite de itens por arquivo).

Referência de aula: [Udemy — MultiResourceItemWriter](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/19975510#overview)

### CompositeItemWriter

Encadeia múltiplos writers, executando-os em sequência para cada lista de itens do chunk.

Referência de aula: [Udemy — CompositeItemWriter](https://www.udemy.com/course/curso-para-desenvolvimento-de-jobs-com-spring-batch/learn/lecture/20017840?start=330#overview)

### ClassifierItemWriter

Seleciona dinamicamente qual writer utilizar com base no tipo ou característica do item.

Projeto pessoal: [sbatch-ecommerce-catalog-aggregator](https://github.com/luisfmaiadc/sbatch-ecommerce-catalog-aggregator)

## Padrão Delegate

O Spring Batch utiliza fortemente o padrão de delegação: cada componente composto ou multi-recurso **delega a responsabilidade real a outro componente especializado**. Exemplos:
- `MultiResourceItemWriter` delega ao writer interno configurado
- `CompositeItemWriter` delega a cada writer da lista

Esse padrão permite reutilizar writers simples dentro de composições complexas sem modificá-los.

---

# 9. Fault Tolerance

## Cenário e Problema

Em Jobs de grande volume, erros pontuais em itens isolados não devem necessariamente interromper o processamento inteiro. O Spring Batch oferece mecanismos de tolerância a falhas para lidar com exceções de forma controlada.

## Skip

O `skip` descarta o item problemático e continua o processamento do chunk.

```java
.faultTolerant()
.skip(Exception.class)
.skipLimit(10)
```

Quando ocorre uma exceção configurada como skip:
- O item é descartado
- O chunk continua
- O contador de skip é incrementado

Se o `skipLimit` for ultrapassado, o Step falha. O Spring Batch **não tenta reprocessar automaticamente** itens skipados.

## Retry

Diferente do skip, o `retry` tenta **reexecutar o item** antes de descartá-lo. É útil para falhas transitórias (ex.: timeout de rede, indisponibilidade momentânea de serviço).

```java
.faultTolerant()
.retry(Exception.class)
.retryLimit(3)
```

---

# 10. ExecutionContext

## Conceito

O `ExecutionContext` é um mapa chave-valor persistido no banco de dados pelo próprio framework, utilizado para armazenar e recuperar estado durante a execução de um Job.

Internamente funciona como:

```java
Map<String, Object>
```

Serve para:
- Armazenar progresso do processamento
- Controlar reinício após falha
- Compartilhar dados entre Steps
- Persistir estado customizado

## Tipos de ExecutionContext

### StepExecutionContext

- Escopo do Step
- Persistido ao final de cada commit
- Ideal para controle de progresso dentro de um Step

### JobExecutionContext

- Escopo do Job inteiro
- Compartilhado entre todos os Steps do Job

## Relação com Restart

Quando um Step falha, o Spring Batch:
1. Lê o `ExecutionContext` salvo no banco
2. Restaura o estado anterior
3. Continua do último checkpoint

Para que o restart funcione corretamente:
- O Reader deve ser restartable
- O estado relevante deve estar sendo salvo no `ExecutionContext`

## Uso Manual

Gravando um valor:

```java
executionContext.put("ultimaPaginaProcessada", 5);
```

Recuperando um valor:

```java
executionContext.getInt("ultimaPaginaProcessada");
```

## Restrições

- Aceita apenas tipos **serializáveis**
- É persistido como `BLOB` no banco de dados
- É salvo automaticamente no commit do chunk

---

# 11. Listeners

## Conceito

Listeners permitem **interceptar eventos do ciclo de vida** do Job ou Step. São úteis para logs, auditoria, métricas, notificações e ajustes dinâmicos de comportamento.

## Implementações Disponíveis

### JobExecutionListener

Intercepta o início e o fim do Job.

```java
beforeJob(JobExecution jobExecution)
afterJob(JobExecution jobExecution)
```

Usos comuns: envio de e-mail ao finalizar, log de início e fim do Job.

### StepExecutionListener

Intercepta o início e o fim de cada Step. Pode inclusive alterar o `ExitStatus` do Step.

```java
beforeStep(StepExecution stepExecution)
afterStep(StepExecution stepExecution)
```

Projeto pessoal: [sbatch-ecommerce-catalog-aggregator](https://github.com/luisfmaiadc/sbatch-ecommerce-catalog-aggregator)

### ChunkListener

Intercepta antes, depois e em caso de erro em cada chunk. Muito útil para métricas de performance por lote.

### ItemReadListener

Intercepta antes da leitura, depois da leitura e em caso de erro na leitura de cada item.

### ItemProcessListener

Intercepta antes do processamento, depois do processamento e em caso de erro no processamento de cada item.

### ItemWriteListener

Intercepta antes da escrita, depois da escrita e em caso de erro na escrita de cada lista de itens.

## Configuração

```java
stepBuilderFactory.get("step")
    .listener(meuListener)
```

## ExecutionContext vs Listener — Guia de Decisão

| Situação | Usar |
| --- | --- |
| Guardar estado para restart | `ExecutionContext` |
| Logar eventos de ciclo de vida | `Listener` |
| Compartilhar dados entre Steps | `JobExecutionContext` |
| Medir tempo de execução | `Listener` |
| Salvar progresso customizado | `ExecutionContext` |

---

# 12. Múltiplos DataSources

Em aplicações com mais de um banco de dados, é necessário:
- Declarar corretamente `jdbcUrl` em cada `DataSource`
- Separar o `DataSource` do negócio do `DataSource` utilizado pelo `JobRepository`

> Manter o `DataSource` do `JobRepository` isolado é opcional, mas recomendado para evitar conflitos de transação entre os metadados do framework e os dados da aplicação.