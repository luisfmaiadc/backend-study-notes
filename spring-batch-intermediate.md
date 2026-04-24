# Otimizações de Jobs com Spring Batch

---

# Profiling

## Cenário e Problema

Antes de aplicar qualquer técnica de otimização — multithread, particionamento ou remote chunking — é necessário **identificar onde está o gargalo**. Sem essa etapa, qualquer intervenção é suposição.

Jobs batch são especialmente suscetíveis a problemas de desempenho porque normalmente:
- Processam grandes volumes de dados
- Executam por longos períodos
- Alocam muitos objetos na heap
- Executam múltiplas transações concorrentes

Se mal dimensionados, podem causar `OutOfMemoryError`, GC excessivo, lock de banco, saturação de CPU e contenção de threads.

## Solução: Profiling como Prática de Engenharia

Profiling é o processo de medir o comportamento real da aplicação durante a execução para identificar gargalos com precisão. Ele também serve para **validar melhorias**: sem medir antes e depois, não há como saber se uma otimização realmente funcionou.

## Principais Métricas

### CPU

Indica processamento pesado, loops ineficientes, algoritmos custosos ou alta concorrência. CPU constantemente próxima de 100% sugere gargalo CPU-bound.

### Memória (Heap)

Indica chunk grande demais, fila de threads superdimensionada, acúmulo de objetos ou vazamento de memória. Sintomas comuns: GC frequente, GC lento e crescimento contínuo da heap.

### Garbage Collection (GC)

GC excessivo pode indicar excesso de criação de objetos, chunks muito grandes, serialização pesada ou acúmulo em filas. GC alto reduz diretamente o throughput.

### Threads

Analise threads bloqueadas, deadlocks, contenção e pool mal dimensionado.

Exemplo clássico de gargalo de infraestrutura:

```
16 threads no pool
5 conexões no pool JDBC
→ 11 threads bloqueadas aguardando conexão
→ O gargalo não é CPU — é infraestrutura
```

### Banco de Dados

Avalie tempo médio de query, locks, espera por conexão e índices ausentes. Muitas vezes o problema não está no Spring Batch — está no SQL.

## Ferramentas de Profiling

| Ferramenta | Tipo | Indicada para |
| --- | --- | --- |
| **VisualVM** | Local / Gratuita | Ambiente local e homologação — CPU, heap, thread dump, hot spots |
| **JConsole** | Local / JDK | Monitoramento básico de heap, threads e GC |
| **Java Flight Recorder (JFR)** | Avançada / JVM | Análise detalhada de CPU, eventos JVM, contenção de locks e alocação de objetos |
| **New Relic / Dynatrace** | APM Corporativo | Monitoramento contínuo em produção, métricas históricas e alertas automáticos |
| **Elastic APM** | APM Open Source | Análise em produção integrada ao ecossistema Elastic |
| **Prometheus + Grafana** | APM Open Source | Dashboards e alertas em ambientes corporativos |

## Sintomas Comuns e Possíveis Causas

| Sintoma | Possível Causa |
| --- | --- |
| CPU 100% constante | Processamento CPU-bound pesado |
| Heap crescendo continuamente | Vazamento de memória ou chunk superdimensionado |
| Muitas threads em estado WAITING | Pool JDBC subdimensionado |
| Lock frequente no banco | Escrita concorrente sem controle adequado |
| Throughput baixo com CPU ociosa | Gargalo de I/O |

## Estratégia de Profiling

1. **Meça antes de otimizar** — nunca aplique multithread, particionamento ou remote chunking sem conhecer o gargalo real.
2. **Reproduza o cenário real** — teste com volume realista e infraestrutura semelhante à produção. Otimizar com 1.000 registros não representa um Job de 10 milhões.
3. **Compare antes e depois** — profiling valida se a otimização foi efetiva.

## Ordem Recomendada de Investigação

Quando um Job está lento, investigue nesta ordem:

1. SQL
2. Índices
3. Pool JDBC
4. Heap e GC
5. CPU
6. Somente então considere paralelismo ou distribuição

---

# 1. Multithreading de Chunks

## Cenário e Problema

Por padrão, um Step baseado em chunk executa de forma **sequencial**: cada chunk é lido, processado, escrito e commitado antes do próximo começar.

```
Read → Process → Write → Commit
Read → Process → Write → Commit
Read → Process → Write → Commit
```

Isso implica que:
- Apenas uma thread executa o Step
- A CPU pode ficar ociosa entre operações
- Recursos da máquina são subutilizados
- Grandes volumes de dados podem levar horas para processar

## Solução: Paralelismo com TaskExecutor

O Multithreading de Chunks permite que **múltiplos chunks sejam processados simultaneamente**, utilizando um pool de threads. Cada chunk é enviado para uma thread independente, cada uma com sua própria transação.

```
Thread 1 → Chunk 1
Thread 2 → Chunk 2
Thread 3 → Chunk 3
Thread 4 → Chunk 4
```

> **Throughput** (ou vazão) é a medida da quantidade de itens que um sistema processa e entrega com sucesso em um dado período de tempo. O paralelismo de chunks aumenta diretamente esse valor.

O paralelismo é habilitado adicionando um `TaskExecutor` ao Step:

```java
.taskExecutor(taskExecutor)
```

Quando isso ocorre:
- Cada chunk é enviado para uma thread do pool
- Múltiplos chunks podem rodar simultaneamente
- Cada chunk possui sua própria transação independente

Fluxo interno:

```
Reader
   ↓
Chunk criado
   ↓
Enviado para TaskExecutor
   ↓
Thread executa: Process → Write → Commit
```

## Configuração do TaskExecutor

A implementação mais comum é o `ThreadPoolTaskExecutor`, um wrapper Spring para o `ThreadPoolExecutor` do Java.

```java
@Bean
public AsyncTaskExecutor taskExecutor() {

    ThreadPoolTaskExecutor executor = new ThreadPoolTaskExecutor();

    executor.setCorePoolSize(8);
    executor.setMaxPoolSize(8);
    executor.setQueueCapacity(10);

    executor.setRejectedExecutionHandler(
        new ThreadPoolExecutor.CallerRunsPolicy()
    );

    return executor;
}
```

Configuração do Step:

```java
.taskExecutor(taskExecutor)
```

### Dimensionamento de Threads

A escolha do número de threads depende do tipo de processamento:

| Tipo de Processamento | Regra Prática |
| --- | --- |
| **CPU Bound** (criptografia, cálculos, regras complexas) | `Threads ≈ Núcleos da CPU + 1` |
| **I/O Bound** (banco de dados, APIs, disco) | `Threads ≈ 2x a 4x o número de núcleos` |

## Funcionamento do ThreadPool

O `ThreadPoolTaskExecutor` possui três componentes principais:

| Componente | Função |
| --- | --- |
| **Core Pool Size** | Threads sempre disponíveis |
| **Queue Capacity** | Fila de chunks aguardando execução |
| **Max Pool Size** | Limite máximo de threads ativas |

### Analogia — Supermercado

| Elemento | Equivalente no ThreadPool |
| --- | --- |
| Caixas abertos | Core Pool |
| Fila de clientes | Queue |
| Caixas extras em horário de pico | Max Pool |

### Ordem de execução

O Java utiliza a seguinte estratégia ao receber uma nova tarefa:

1. Preenche o **Core Pool** (threads imediatas)
2. Se lotado, enfileira na **Queue**
3. Se a Queue estiver cheia, cria threads até o **Max Pool**
4. Se tudo estiver cheio, aplica a **política de rejeição**

> Criar threads é caro (CPU + memória). Enfileirar tarefas é barato. Por isso a Queue é preenchida antes do Max Pool ser atingido.

## Relação entre Chunk Size e Memória

Cada chunk carrega uma lista de itens na memória. A combinação de tamanho do chunk e capacidade da fila determina o consumo de memória do Step.

Exemplo:

```
chunk(500) + QueueCapacity = 10
→ 10 * 500 = 5.000 objetos aguardando na fila
+ objetos dos chunks já em execução
```

Quanto maior o chunk e a fila, maior o risco de `OutOfMemoryError`. Calibre esses valores de acordo com a memória disponível na JVM.

## Backpressure (Contrapressão)

Quando o Core Pool, a Queue e o Max Pool estão todos saturados, o executor aplica a política `CallerRunsPolicy`: a thread que estava submetendo tarefas passa a executar o chunk diretamente.

Esse mecanismo é chamado de **Backpressure** (Contrapressão). Ele:
- Desacelera a leitura de novos chunks
- Reduz a pressão sobre o sistema
- Evita que o produtor gere trabalho mais rápido do que o consumidor processa

## Thread Safety

Nem todos os componentes do Spring Batch são thread-safe. Em Steps multithreaded, isso é crítico.

### ItemReader

Readers como `FlatFileItemReader` e `JdbcCursorItemReader` **não são thread-safe**. A solução é envolvê-los com:

```java
SynchronizedItemStreamReader
```

Ele garante que apenas uma thread execute a leitura por vez.

### ItemWriter

`FlatFileItemWriter` também não é thread-safe e pode corromper arquivos se múltiplas threads escreverem simultaneamente.

Writers seguros para uso concorrente:

```java
JdbcBatchItemWriter  // o banco gerencia a concorrência
```

## Transações, Ordem e Idempotência

### Transações independentes por chunk

Com paralelismo, a regra `1 Chunk = 1 Transação` é mantida:

```
Thread 1 → Chunk 1 → Commit
Thread 2 → Chunk 2 → Commit
Thread 3 → Chunk 3 → Commit
```

Se ocorrer erro, o rollback afeta apenas o chunk da thread que falhou.

### Ordem de processamento

| Etapa | Ordem |
| --- | --- |
| Leitura | Mantida |
| Processamento | Paralelo (sem garantia de ordem) |
| Escrita | Não garantida |

`Chunk 3 pode terminar antes do Chunk 1`. Se a lógica depender de ordem, multithreading pode quebrar o sistema.

### Restart e Idempotência

Chunks já commitados não são revertidos em caso de falha. O Job reinicia do próximo checkpoint. Para evitar duplicação de dados, o Writer deve ser **idempotente**:

```sql
INSERT com chave única
UPSERT
MERGE
```

## Alinhamento: ThreadPool vs Pool de Conexões

Um erro comum é configurar mais threads do que conexões JDBC disponíveis:

```
16 threads
5 conexões no pool JDBC
→ Threads ficam bloqueadas esperando conexão
→ CPU ociosa, perda de performance
```

Sempre alinhe:
- Tamanho do ThreadPool
- Tamanho do Connection Pool
- Capacidade de concorrência do banco

## Quando Utilizar

- Grandes volumes de dados (ex.: 1 milhão de registros)
- Processamento I/O Bound (banco, API, disco, rede)
- Escritas em banco que suportam concorrência
- Transformações de itens independentes entre si (clientes, logs, arquivos, métricas)

## Quando Não Utilizar

- Quando há dependência de ordem entre itens (ex.: arquivo sequencial, processamento encadeado)
- Quando recursos utilizados não são thread-safe (ex.: `FlatFileItemWriter` sem sincronização)
- Quando o banco possui baixa capacidade de concorrência e pode se tornar gargalo de lock

## Projeto de Referência

Implementação prática desta técnica: [sbatch-employee-importer](https://github.com/luisfmaiadc/sbatch-employee-importer)

---

# 2. Steps Paralelos (Parallel Steps)

## Cenário e Problema

O Multithread Step paraleliza chunks dentro de um único Step. Mas quando o Job possui **múltiplos Steps independentes** que poderiam rodar ao mesmo tempo, executá-los em sequência desperdiça tempo e recursos.

| Técnica | Paralelismo |
| --- | --- |
| **Multithread Step** | Dentro de um único Step (chunks em paralelo) |
| **Steps Paralelos** | Entre Steps diferentes (fluxos simultâneos) |

## Solução: Flows com `.split()` e TaskExecutor

Para paralelizar Steps, utilizamos o conceito de **Flow**.

### O que é um Flow?

Um Flow é um bloco de execução capaz de conter:
- Um único Step
- Uma sequência inteira de Steps
- Regras condicionais (sucesso, falha, transições)

Steps Paralelos = execução simultânea de **Flows**, não de Steps isolados.

### Estrutura base

```java
return new FlowBuilder<SimpleFlow>("jobFlow")
    .start(flow1)
    .split(taskExecutor)
    .add(flow2, flow3)
    .build();
```

- `.split()` — divide a execução em múltiplas threads
- `.add()` — define quais Flows rodam em paralelo
- `TaskExecutor` — controla quantas threads estarão disponíveis

> O paralelismo máximo é limitado pelo tamanho do pool de threads. Com 3 Flows e 2 threads, apenas 2 Flows rodam simultaneamente; o terceiro aguarda.

### Flows com múltiplos Steps

Flows podem conter sequências internas:

- Flow 1 → Step A → Step B
- Flow 2 → Step C

Os Flows rodam em paralelo entre si, mas os Steps dentro de cada Flow executam sequencialmente.

### Continuação após o bloco paralelo

```java
.next(stepFinal)
```

O `stepFinal` só executa após **todos os Flows terminarem** (comportamento de JOIN/barreira de sincronização).

### Tempo total do Job

```
Tempo total = duração do Flow mais lento
```

Exemplo: Flow 1 = 10 min, Flow 2 = 2 min → Job termina em 10 min.

## Cuidados Importantes

### Race Condition no JobExecutionContext

O `JobExecutionContext` é memória compartilhada. Se múltiplos Flows leem e gravam os mesmos dados, pode ocorrer Race Condition com dados inconsistentes ou sobrescritos.

Steps Paralelos exigem **isolamento lógico e de estado entre Flows**.

### Comportamento em caso de falha

Se qualquer Flow falhar:
- O Job termina com status `FAILED`
- Os outros Flows **não são cancelados automaticamente** — o Spring Batch aguarda todos terminarem antes de marcar o Job como `FAILED`

## Quando Utilizar

- Pipelines completamente independentes (ex.: Step A processa clientes, Step B processa pedidos, Step C processa logs)
- Steps sem dependência de ordem entre si
- Steps que não compartilham estado ou contexto
- O gargalo está na execução sequencial de múltiplos pipelines

## Quando Não Utilizar

- Steps que dependem do resultado uns dos outros
- Steps que compartilham variáveis ou contexto mutável
- Steps com necessidade de ordenação
- Steps que processam o mesmo conjunto de dados

Nesses casos, prefira **Multithread Step** ou **Particionamento**.

## Projeto de Referência

Implementação prática desta técnica: [sbatch-ecommerce-catalog-aggregator](https://github.com/luisfmaiadc/sbatch-ecommerce-catalog-aggregator)

---

# 3. Processamento Assíncrono (Async Processing)

## Cenário e Problema

Quando o gargalo do Step está no `ItemProcessor` — especialmente em operações I/O-bound como chamadas REST, integrações com APIs ou enriquecimento remoto de dados —, o processamento sequencial item a item gera ociosidade na thread principal enquanto aguarda respostas externas.

## Solução: AsyncItemProcessor e AsyncItemWriter

O Async Processing delega o processamento para um pool de threads. A thread principal continua lendo novos itens enquanto as threads do pool executam o processamento em paralelo.

**Componentes necessários:**
- `AsyncItemProcessor` — delega o processamento ao pool e retorna um `Future`
- `AsyncItemWriter` — aguarda os `Future`s e encaminha os resultados para um Writer real
- `TaskExecutor` — pool de threads que executa o processamento

> O Async Processing utiliza **reuso de threads** (pool), não criação de uma nova thread por item.

### Mudança no contrato do Processor

| Processor Normal | AsyncItemProcessor |
| --- | --- |
| `Item → ItemProcessado` | `Item → Future<ItemProcessado>` |

O retorno não é o dado pronto, mas uma promessa de que ele estará disponível no futuro.

## Fluxo de Execução

### Thread Principal (Master)

1. Lê o item (Reader)
2. Envia o item ao `AsyncItemProcessor`
3. Recebe imediatamente um `Future`
4. Continua lendo novos itens **sem bloquear**

### Threads do Pool

- Executam o processamento real
- Realizam chamadas externas (REST, RPC etc.)
- Resolvem o `Future` com o resultado

### AsyncItemWriter como ponto de sincronização

O `AsyncItemWriter` **não grava dados diretamente**. Ele:

1. Aguarda todos os `Future`s do chunk serem resolvidos
2. Extrai os objetos reais
3. Encaminha para um Writer real (`JdbcBatchItemWriter`, `FlatFileItemWriter` etc.)

Isso preserva a **integridade transacional do chunk**: o Spring Batch só escreve quando todos os Futures estão resolvidos. Em caso de erro, o rollback afeta o lote inteiro.

## Cuidados Importantes

### Gargalo no item mais lento (Efeito Comboio)

O `AsyncItemWriter` funciona como uma barreira: aguarda o item mais lento do chunk antes de liberar a escrita.

Exemplo:
```
Chunk = 1.000 itens
Item 1 = 10 segundos
Itens 2–1.000 = 10 ms
→ Todo o chunk fica parado aguardando o Item 1
```

O desempenho final é limitado pelo **pior caso do lote**.

### Impacto na Thread Principal

Se o Writer estiver bloqueado aguardando Futures:
- O Reader para
- O próximo chunk não é lido
- A thread principal fica bloqueada

O processamento é paralelo, mas o controle do Step **não é totalmente assíncrono** de ponta a ponta.

## Quando Utilizar

- Chamadas REST e integrações com APIs externas
- Enriquecimento remoto de dados
- Operações com alta latência variável
- O ItemProcessor é mais lento que Reader e Writer

## Quando Não Utilizar

- O gargalo está no banco de dados ou no Writer
- O processamento é CPU-bound pesado
- Itens possuem latência muito desigual (risco do Efeito Comboio)
- Chunk muito grande combinado com item ocasionalmente lento

## Projeto de Referência

Implementação prática desta técnica: [sbatch-async-rest-order-enrichment](https://github.com/luisfmaiadc/sbatch-async-rest-order-enrichment)

---

# 4. Particionamento

## Visão Geral

Para minimizar gargalos de I/O em grandes volumes de dados, o Spring Batch permite dividir um Step em **múltiplas partições** processadas em paralelo por Workers independentes.

Cada Worker é um Step completo, contendo:
- `ItemReader`
- `ItemProcessor`
- `ItemWriter`

Por serem Steps independentes, cada partição mantém seu próprio estado no `JobRepository`, o que preserva a **restartabilidade** do Job.

## Funcionamento Interno

O particionamento é composto por três elementos:

### Master Step

Responsável por:
- Dividir o trabalho em partições via `Partitioner`
- Delegar a execução aos Workers
- Controlar a agregação do resultado final

O Master **não executa processamento de dados diretamente**.

### Partitioner

Implementa a lógica de divisão de dados. Recebe o `gridSize` e retorna um `Map<String, ExecutionContext>`, onde cada `ExecutionContext` contém os parâmetros da fatia de dados que o Worker irá processar.

Exemplos de estratégias de divisão:
- Intervalos de IDs (ex.: 1–1000, 1001–2000…)
- Faixas de datas
- Arquivos diferentes
- Segmentos de tabela

O `gridSize` define o número de partições, mas não o número exato de threads — isso depende do executor configurado.

### PartitionHandler

Define **como** os Workers serão executados. Principais implementações:

| Implementação | Descrição |
| --- | --- |
| `TaskExecutorPartitionHandler` | Executa Workers localmente, na mesma JVM, via TaskExecutor |
| `MessageChannelPartitionHandler` | Executa Workers em JVMs remotas via mensageria |
| `DeployerPartitionHandler` | Usa Spring Cloud para criar Workers sob demanda |

---

## Particionamento Local

No Particionamento Local, todos os Workers rodam na mesma JVM, usando o `TaskExecutorPartitionHandler`. A paralelização ocorre via threads de um `ThreadPoolTaskExecutor`.

### Quando Utilizar

- Processamento de grandes volumes em uma única máquina (milhões de registros, grandes arquivos CSV)
- Atualizações por faixa de ID (ex.: reprocessar dados históricos, atualizar status de pedidos em lote)
- Fechamento por período (ex.: dividir por intervalo de datas)
- Quando não há necessidade de arquitetura distribuída e o servidor já possui múltiplos núcleos

### Quando Não Utilizar

- Volumes pequenos onde o overhead de particionamento não se justifica
- Quando há forte dependência entre registros
- Quando o gargalo está na CPU e não em I/O
- Quando o banco não suporta alto nível de concorrência

### Cuidados Importantes

#### Banco de Dados Compartilhado

No particionamento local, Master e Workers compartilham o mesmo `JobRepository`. O banco deve:
- Suportar concorrência adequada
- Ter índices nas tabelas do `JobRepository` para evitar gargalos de lock

#### Definição do gridSize

| Cenário | Recomendação |
| --- | --- |
| CPU Bound | `gridSize` próximo ao número de núcleos |
| I/O Bound | `gridSize` pode ser maior que o número de núcleos |

Sempre considere: conexões disponíveis no pool, limite do banco e memória da JVM.

#### Thread Safety dos Componentes

Cada Worker possui sua própria instância de Reader/Processor/Writer. Ainda assim, evite:
- Uso de variáveis estáticas
- Compartilhamento manual de estado entre Workers

#### Estratégia de divisão

O particionamento só faz sentido quando:
- É possível dividir os dados em partes **independentes**
- Não há dependência entre registros
- Não há necessidade de ordenação global durante o processamento

### Projeto de Referência

Implementação prática desta técnica: [sbatch-partitioner-score-update](https://github.com/luisfmaiadc/sbatch-partitioner-score-update)

---

## Particionamento Remoto

No Particionamento Remoto, o Step Master (Manager) delega o processamento para Workers executando em **outras JVMs**, possivelmente em outras máquinas. A comunicação ocorre através de um **Message Broker**.

Brokers comumente utilizados:
- RabbitMQ
- ActiveMQ
- Kafka

A execução deixa de ser baseada em threads locais e passa a ser baseada em **mensageria distribuída**.

### Arquitetura

#### Manager (Master)

Responsável por:
- Criar as partições via `Partitioner`
- Enviar cada `ExecutionContext` para o broker
- Aguardar a finalização das partições
- Consolidar o resultado final

O Manager **não processa dados diretamente**.

#### Workers

Cada Worker:
- Escuta mensagens do broker
- Recebe uma partição
- Executa Reader, Processor e Writer
- Atualiza seu estado no `JobRepository`
- Retorna o status ao Manager

Cada Worker é uma **aplicação Spring Batch independente**.

#### Message Broker

Responsável por:
- Garantir entrega das mensagens
- Controlar filas e distribuir carga entre Workers
- Permitir escalabilidade horizontal
- Desacoplar Manager e Workers

### Implementação

Para implementar o Particionamento Remoto é necessário:
- Customizar um `Partitioner` para gerar as partições e seus `ExecutionContext`
- Utilizar o `MessageChannelPartitionHandler` (em vez do `TaskExecutorPartitionHandler`)
- Configurar canais de envio e recebimento de mensagens via Spring Integration

### Banco de Dados e Restart

Manager e Workers devem compartilhar o **mesmo JobRepository** e apontar para o **mesmo banco de dados**. Isso é essencial para:
- Garantir consistência de estado entre instâncias
- Permitir restart correto em caso de falha
- Evitar reprocessamento indevido

### Quando Utilizar

- Quando uma única máquina não suporta a carga (CPU, memória ou I/O se tornam gargalos)
- Volumes massivos (reprocessamento histórico, fechamentos noturnos de grande escala)
- Quando é necessário escalar horizontalmente adicionando Workers dinamicamente
- Gargalo em I/O distribuído (múltiplos bancos, integrações externas, escrita em sistemas distintos)

### Quando Não Utilizar

- Quando uma única máquina já atende a demanda
- Quando o gargalo não é I/O
- Quando a infraestrutura não comporta mensageria
- Quando o custo operacional não se justifica

### Cuidados Importantes

#### Ordem de inicialização

Os Workers devem estar ativos **antes** do Manager iniciar o envio das partições. Caso contrário:
- Mensagens podem se acumular no broker
- Pode ocorrer timeout ou falha de execução

#### Latência de rede

Particionamento remoto só compensa quando a latência é baixa e o custo de comunicação é menor que o ganho de paralelismo. Do contrário, o overhead de mensageria pode anular o benefício.

#### Idempotência

Como há comunicação distribuída, mensagens podem ser reenviadas e Workers podem falhar no meio do processamento. O processamento deve ser **idempotente**.

#### Complexidade operacional

Particionamento remoto adiciona: broker, monitoramento distribuído, deploy de múltiplas aplicações e observabilidade mais complexa. É uma solução arquitetural — use quando a necessidade for real.

### Comparação: Particionamento Local vs Remoto

| Aspecto | Local | Remoto |
| --- | --- | --- |
| Execução | Mesma JVM | Múltiplas JVMs |
| Comunicação | Threads | Mensageria |
| Escalabilidade | Vertical | Horizontal |
| Complexidade | Média | Alta |
| Infraestrutura | Simples | Broker + múltiplas apps |
| Indicado para | Carga moderada | Carga massiva |

---

### Comparação: Multithread Step vs Particionamento

| Multithread Step | Particionamento |
| --- | --- |
| Um único Step | Master + múltiplos Workers |
| Compartilha Reader | Cada Worker tem seu próprio Reader |
| Mais simples de configurar | Mais escalável |
| Menor isolamento por chunk | Maior isolamento por partição |

Particionamento é mais robusto quando o objetivo é **escalar processamento mantendo restartabilidade forte e isolamento por partição**.

---

# 5. Remote Chunking

## Cenário e Problema

O Remote Chunking é voltado para cenários onde o **gargalo do Job está no processamento (CPU-bound)** e não na leitura ou escrita. Enquanto no Particionamento Remoto cada Worker executa um Step completo (Reader + Processor + Writer), no Remote Chunking a responsabilidade é dividida de forma diferente.

| Técnica | Responsabilidade do Manager | Responsabilidade dos Workers |
| --- | --- | --- |
| **Particionamento Remoto** | Divide dados em partições | Executa Step completo (Reader + Processor + Writer) |
| **Remote Chunking** | Executa o Reader e forma chunks | Executa Processor + Writer por chunk recebido |

## Solução: Arquitetura Manager + Workers via Broker

### Manager

Responsável por:
- Executar o `ItemReader`
- Agrupar itens em chunks
- Enviar os chunks ao Message Broker
- Aguardar o retorno de status de cada chunk

O Manager **não executa processamento nem escrita**.

### Workers

Cada Worker:
- Recebe um chunk via mensagem
- Executa o `ItemProcessor`
- Executa o `ItemWriter`
- Retorna status de sucesso ou falha

Os Workers **não possuem Reader**.

### Message Broker

Brokers suportados:
- RabbitMQ
- ActiveMQ
- Kafka

Responsabilidades do broker:
- Entrega e persistência das mensagens
- Distribuição de carga entre Workers
- Suporte à política de redelivery

## Fluxo de Execução

1. Manager lê os itens
2. Forma um chunk
3. Envia o chunk para o broker
4. Worker consome o chunk
5. Worker processa e escreve
6. Worker envia resposta ao Manager
7. Manager controla commit e progresso

> O commit continua sendo baseado em chunk, mas a execução ocorre remotamente.

## Implementação

Para implementar Remote Chunking é necessário:
- Configurar canais de saída (Manager → Broker)
- Configurar canais de entrada (Worker → Broker)
- Definir `IntegrationFlows` via Spring Integration
- Configurar mecanismos de serialização dos itens

Diferente do Particionamento Remoto:
- Não é necessário criar um `Partitioner`
- Não há `gridSize`
- Não há divisão por faixa de dados

Qualquer Worker disponível pode consumir qualquer chunk.

## Cuidados Importantes

### Latência de Rede

Remote Chunking só compensa quando:
- A latência de rede é baixa
- O custo de serialização + envio for menor que o ganho de paralelismo

Se não for o caso, pode ser mais lento que o processamento local.

### Garantia de Entrega

O Spring Batch **não armazena o conteúdo do chunk nos metadados do JobRepository**. Se um Worker cair durante o processamento, o framework precisa reenviar o chunk. Para isso, configure o broker com:
- Persistência de mensagens
- Acknowledgment adequado
- Política de redelivery

### Tamanho do Chunk

| Chunk pequeno | Chunk grande |
| --- | --- |
| Excesso de mensagens no broker | Alto consumo de memória nos Workers |
| Overhead de rede | Tempo alto de processamento antes do commit |

Balanceie: custo de comunicação, tempo de processamento e capacidade dos Workers.

### Serialização

Os itens precisam ser serializáveis. Evite:
- Objetos pesados
- Payloads com dados desnecessários
- Dependência de estado local

## Restartabilidade

O controle de restart é feito pelo Manager. Em caso de falha:
- Chunks não confirmados podem ser reenviados
- Workers devem ser **idempotentes**
- Escritas devem tolerar reprocessamento

Boas práticas:
- Chaves únicas no banco
- Operações UPSERT
- Processamento sem efeitos colaterais externos

## Quando Utilizar

- O gargalo é CPU (criptografia, cálculos complexos, enriquecimento com regras pesadas, transformações intensivas)
- A leitura é rápida, mas o processamento é lento
- É necessário escalar horizontalmente **apenas o processamento**, mantendo leitura centralizada

## Quando Não Utilizar

- O gargalo está em I/O (leitura ou escrita lenta)
- A infraestrutura de mensageria não está madura
- O processamento é leve e não justifica a complexidade

---

# 6. Comparação entre Técnicas de Performance

| Técnica | Escala o quê? | Resolve qual gargalo? | Complexidade | Infraestrutura extra | Quando usar | Quando evitar |
| --- | --- | --- | --- | --- | --- | --- |
| **Ajuste de Chunk Size** | Throughput por transação | Overhead de commit | ⭐ Baixa | Não | Quando precisa melhorar performance sem mudar arquitetura | Quando gargalo é CPU ou I/O pesado |
| **JdbcBatchItemWriter** | Escrita em banco | I/O de escrita | ⭐ Baixa | Não | Quando INSERT/UPDATE é gargalo | Quando problema está no processamento |
| **Paginação (vs Cursor)** | Consumo de memória | Uso excessivo de RAM | ⭐ Baixa | Não | Grandes volumes de dados | Quando latência de múltiplas queries prejudica |
| **Multithread Step** | Execução de chunks | I/O moderado | ⭐⭐ Média | Não | Paralelismo simples na mesma JVM | Quando precisa isolamento forte |
| **Particionamento Local** | Step inteiro | I/O alto | ⭐⭐⭐ Média | Não | Grande volume em uma única máquina | Quando há dependência entre registros |
| **Particionamento Remoto** | Step inteiro distribuído | I/O massivo | ⭐⭐⭐⭐ Alta | Message Broker + múltiplas JVMs | Quando uma máquina não suporta a carga | Quando rede é lenta ou instável |
| **Remote Chunking** | Processamento (CPU) | CPU-bound | ⭐⭐⭐⭐ Alta | Message Broker | Processamento pesado, leitura rápida | Quando gargalo é leitura ou escrita |
| **Banco Tunado (Índices/Pool)** | I/O banco | Lock, lentidão SQL | ⭐⭐ Média | Ajuste de infraestrutura | Sempre que houver alto volume | Quando problema é CPU |
| **Escala Vertical** | CPU + RAM | Limitação física | ⭐ Baixa | Upgrade de servidor | Quando precisa de solução rápida | Quando arquitetura já é gargalo |
| **Escala Horizontal** | Capacidade total | Limite de máquina única | ⭐⭐⭐⭐ Alta | Orquestração distribuída | Sistemas críticos de alto volume | Quando custo não compensa |

## Leitura Estratégica da Tabela

Antes de aplicar qualquer técnica, responda:

### Onde está o gargalo?

- CPU?
- Escrita em banco?
- Leitura?
- Rede?
- Memória?
- Lock de tabela?

O erro clássico é aplicar paralelismo onde o problema real é SQL mal otimizado.

### O problema é arquitetural ou de configuração?

- Índice faltando → não é problema do Spring Batch
- Pool JDBC mal dimensionado → não é problema de chunk
- Banco subdimensionado → não é problema de paralelismo

### Ordem recomendada de otimização

1. SQL
2. Índices
3. Pool de conexões
4. Chunk size
5. Multithread Step
6. Particionamento Local
7. Particionamento Remoto / Remote Chunking

Avançar para arquitetura distribuída sem esgotar as etapas anteriores é um erro de engenharia.