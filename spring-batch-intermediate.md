# Multithreading de Chunks no Spring Batch

## 1. Objetivo do Multithreading

Por padrão, um Step baseado em chunk executa de forma sequencial:

```
Read → Process → Write → Commit
Read → Process → Write → Commit
Read → Process → Write → Commit
```

Cada chunk é executado um após o outro. Isso significa que:
- apenas **uma thread** executa o Step
- a CPU pode ficar ociosa
- recursos da máquina são subutilizados

O Multithreading de Chunks permite que **vários chunks sejam processados simultaneamente**, utilizando múltiplas threads.

Fluxo com paralelismo:

```
Thread 1 → Chunk 1
Thread 2 → Chunk 2
Thread 3 → Chunk 3
Thread 4 → Chunk 4
```

Isso aumenta drasticamente o *throughput* do Job.

📌 **Throughput** (ou vazão) é a medida da quantidade de itens de trabalho, produtos ou dados que um sistema processa e entrega com sucesso em um período de tempo específico.

## 2. Quando Utilizar Multithreading

Multithreading é recomendado quando:

### ✔ Grandes volumes de dados

Exemplo:

```
1 milhão de registros
```

Processamento sequencial pode levar horas.

### ✔ Processamento I/O Bound

Quando o sistema **fica esperando recursos externos:**
- Banco de dados
- API externa
- Disco
- Rede

Enquanto uma thread espera I/O, outra pode trabalhar.

### ✔ Escritas em banco em lote

Se o banco suporta concorrência, múltiplas threads podem inserir dados simultaneamente.

### ✔ Transformações independentes

Quando os itens não possuem dependência entre si.

Exemplo:
- Importar clientes
- Processar logs
- Converter arquivos
- Calcular métricas

## 3. Quando NÃO Utilizar

Evite multithreading quando existir:

### ❌ Dependência de ordem

Exemplo:
- geração de arquivo sequencial
- processamento dependente do item anterior

### ❌ Recursos não thread-safe

Exemplo:

```java
FlatFileItemWriter
```

Sem sincronização adequada pode corromper arquivo.

### ❌ Banco com baixa capacidade de concorrência

Alguns bancos podem se tornar gargalo de lock.

## 4. Como o Multithreading Funciona no Spring Batch?

O paralelismo é habilitado adicionando um `TaskExecutor` ao Step.

Exemplo:

```java
.taskExecutor(taskExecutor)
```

Quando isso ocorre:
- cada chunk é enviado para uma thread
- múltiplos chunks podem rodar simultaneamente
- cada chunk possui sua própria transação

Fluxo:

```
Reader
   ↓
Chunk criado
   ↓
Enviado para TaskExecutor
   ↓
Thread executa:
Process → Write → Commit
```

## 5. TaskExecutor

O `TaskExecutor` é o componente responsável por gerenciar as threads.

A implementação mais comum é:

```java
ThreadPoolTaskExecutor
```

Ele é um wrapper Spring para o **ThreadPoolExecutor do Java.**

### Exemplo de Configuração

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

Depois ele é utilizado no Step:

```java
.taskExecutor(taskExecutor)
```

## 6. Estrutura e funcionamento do ThreadPool

O `ThreadPoolTaskExecutor` possui três componentes principais:

| Componente     | Função                             |
| -------------- | ---------------------------------- |
| Core Pool Size | Threads sempre disponíveis         |
| Queue Capacity | Fila de chunks aguardando execução |
| Max Pool Size  | Limite máximo de threads           |

### Analogia – Supermercado 🛒

Imagine um supermercado:

| Elemento                         | Equivalente |
| -------------------------------- | ----------- |
| Caixas abertos                   | Core Pool   |
| Fila de clientes                 | Queue       |
| Caixas extras em horário de pico | Max Pool    |

Fluxo:

```
Clientes chegam
↓
Se há caixa livre → atendimento imediato
↓
Se todos ocupados → cliente entra na fila
↓
Se fila cheia → abre novo caixa
```

### Ordem de Execução do ThreadPool

O Java utiliza a seguinte estratégia:

1️⃣ Preenche o **Core Pool**

2️⃣ Depois enche a **Queue**

3️⃣ Depois cria threads até o **Max Pool**

4️⃣ Se tudo estiver cheio → executa **política de rejeição**

Motivo:
- Criar threads é caro (CPU + memória).
- Enfileirar tarefas é barato.

## 7. Relação entre Chunk Size e Memória

Cada chunk contém uma **lista de itens na memória.**

Exemplo:

```java
chunk(500)
```

Significa:

```
500 objetos na memória por chunk
```

Agora imagine:

```
QueueCapacity = 10
```

Então teremos:

```
10 chunks aguardando execução
```

Consumo aproximado:

```
10 * 500 = 5000 objetos
```

Mais:

```
+ objetos dos chunks já executando
```

Quanto maior o chunk e a fila:
- maior uso de memória
- maior risco de `OutOfMemoryError`

## 8. Benefício da Fila

Uma fila pequena traz benefício importante:
- Threads nunca ficam ociosas

Assim que uma termina:
- já existe um chunk esperando

Isso mantém a CPU sempre ocupada.

## 9. O que acontece quando a fila enche?

Quando:
- Core Pool cheio
- Queue cheia
- Max Pool atingido

O executor aplica a política:

```
CallerRunsPolicy
```

Isso significa:

    A thread que estava submetendo tarefas executa o chunk.

Efeito:
- desacelera a leitura
- reduz pressão no sistema

Esse mecanismo é chamado de:

### 📌 Backpressure (Contrapressão)

Ele evita que o sistema produza trabalho mais rápido do que consegue processar.

## 10. Configuração de Threads

A escolha depende do tipo de processamento.

### CPU Bound

Processamento pesado de CPU:
- criptografia
- cálculos
- regras complexas

Regra aproximada:

```
Threads ≈ Núcleos da CPU + 1
```

### I/O Bound

Processamento que espera recursos externos:
- banco de dados
- APIs
- disco

Regra aproximada:

```
Threads ≈ 2x a 4x número de núcleos
```

## 11. Thread Safety (Muito Importante)

Nem todos os componentes do Spring Batch são thread-safe.

Em Step multithread isso é crítico.

### ItemReader

Muitos readers não são thread-safe.

Exemplo:

```
FlatFileItemReader
JdbcCursorItemReader
```

Solução:

```
SynchronizedItemStreamReader
```

Ele garante que apenas uma thread leia por vez.

### ItemWriter

Alguns writers também não são seguros.

Exemplo:

```
FlatFileItemWriter
```

Pode corromper arquivos se múltiplas threads escreverem simultaneamente.

#### Writers seguros

Exemplo seguro:

```
JdbcBatchItemWriter
```

Porque o banco gerencia concorrência.

## 12. Transações no Step Multithread

Mesmo com paralelismo:

A regra continua sendo:

```
1 Chunk = 1 Transação
```

Exemplo:

```
Thread 1 → Chunk 1 → Commit
Thread 2 → Chunk 2 → Commit
Thread 3 → Chunk 3 → Commit
```

Cada thread possui **transação independente.**

Se ocorrer erro:
- Rollback afeta apenas o chunk da thread que falhou.

## 13. Ordem de Processamento

Com multithreading:

| Etapa         | Ordem         |
| ------------- | ------------- |
| Leitura       | Mantida       |
| Processamento | Paralelo      |
| Escrita       | Não garantida |

Isso significa que:

```
Chunk 3 pode terminar antes do Chunk 1
```

Se a lógica depender de ordem, multithreading pode quebrar o sistema.

## 14. Restart e Idempotência

Se ocorrer falha:
- chunks já commitados **não são revertidos**
- o Job reinicia do próximo checkpoint

Por isso é essencial garantir:

### Writer idempotente

Exemplo:

```sql
INSERT com chave única
UPSERT
MERGE
```

Isso evita duplicação de dados.

## 15. ThreadPool vs Pool de Conexões

Erro comum em sistemas batch:

```
16 threads
5 conexões no pool JDBC
```

Resultado:
- threads ficam bloqueadas esperando conexão
- CPU ociosa
- perda de performance

Sempre alinhar:
- ThreadPool
- Connection Pool
- Capacidade do banco

## 16. 🧠 Resumo Mental

Fluxo do Step multithread:

```
Reader
   ↓
Chunks criados
   ↓
TaskExecutor
   ↓
Threads executam em paralelo
   ↓
Process → Write
   ↓
Commit por chunk
```

### Conclusão

Multithreading de chunks permite:
- aumentar throughput
- utilizar melhor CPU
- reduzir tempo de execução de Jobs

Porém exige atenção em:
- thread safety
- memória
- concorrência no banco
- ordem de processamento
- idempotência

Quando bem configurado, pode reduzir execução de Jobs de:

```
3 horas → poucos minutos
```

# Steps Paralelos no Spring Batch (Parallel Steps)

## 1. Diferença fundamental: Multithread Step vs Steps Paralelos

### Multithread Step
- Paralelismo dentro de um único Step
- Várias threads processam chunks do mesmo step

### Steps Paralelos (Parallel Steps)
- Paralelismo entre Steps diferentes
- Steps (ou fluxos de steps) executam simultaneamente

📌 Use Steps Paralelos quando **os Steps são independentes entre si.**

## 2. Quando usar Steps Paralelos?

A técnica é ideal quando:
- Os Steps **não compartilham dados**
- Não há dependência de ordem entre Steps
- Cada Step processa **dados diferentes ou domínios diferentes**
- O gargalo está na **execução de múltiplos pipelines**, não em um único Step

Exemplos práticos:
- Step A processa clientes
- Step B processa pedidos
- Step C processa logs

   → Todos podem rodar em paralelo

## 3. Objetivo principal

Maximizar:
- Throughput (vazão)
- Uso de CPU e I/O
- Tempo total do Job (redução de duração)

📌 Em vez de executar Steps em sequência, o Job **executa múltiplos pipelines ao mesmo tempo.**

## 4. Conceito central: Flow

Para paralelizar Steps, usamos **Flow**.

### O que é um Flow?

Um Flow é um mini-job, capaz de conter:
- Um único Step
- **Uma sequência inteira de Steps**
- Regras condicionais (sucesso, falha, transições)

📌 Steps Paralelos = execução paralela de **Flows**, não apenas Steps isolados.

## 5. Como o paralelismo é criado? (split + TaskExecutor)

O paralelismo acontece usando:
- `FlowBuilder`
- `.split(TaskExecutor)`
- `.add(flow1, flow2, ...)`

Estrutura base:

```java
return new FlowBuilder<SimpleFlow>("jobFlow")
    .start(flow1)
    .split(taskExecutor)
    .add(flow2, flow3)
    .build();
```

### Significado:
- `.split()` → divide a execução em múltiplas threads
- `.add()` → define quais Flows rodam em paralelo
- `TaskExecutor` → controla quantas threads estarão disponíveis

## 6. TaskExecutor limita o paralelismo real

Se:
- Você adiciona 3 flows
- Mas o executor tem 2 threads

👉 Apenas **2 flows rodam ao mesmo tempo**

👉 O terceiro **fica aguardando thread livre**

📌 Paralelismo máximo depende diretamente do tamanho do pool de threads.

## 7. Flows podem conter múltiplos Steps

Exemplo:
- Flow 1 → Step A → Step B
- Flow 2 → Step C

Eles executam:
- Em paralelo entre Flows
- Mas sequencialmente **dentro do mesmo Flow**

📌 Cada Flow é uma trilha independente.

## 8. Independência entre Flows é obrigatória

Flows paralelos:
- **NÃO devem depender um do outro**
- **NÃO devem compartilhar estado mutável**

Caso contrário, surgem problemas de concorrência.

## 9. Ponto crítico: JobExecutionContext e Race Condition

O `JobExecutionContext` é memória compartilhada.

Se múltiplos Flows:
- leem
- alteram
- gravam os mesmos dados

👉 Pode ocorrer Race Condition

👉 Dados inconsistentes ou sobrescritos

📌 Steps Paralelos exigem **isolamento lógico e de estado.**

## 10. Sincronização: o Job espera todos os Flows terminarem

Após o `.split()`:
- O Job **aguarda TODOS os flows concluírem**
- Funciona como um **JOIN (barreira de sincronização)**

📌 O próximo Step só começa quando **o último Flow terminar.**

## 11. Continuação após o bloco paralelo (`.next()`)

Após os Flows paralelos:

```java
.next(stepFinal)
```

👉 O stepFinal, se houver, **roda somente após todos os flows terminarem.**

## 12. Tempo total do Job em paralelo

O tempo total passa a ser:

```
Tempo = duração do Flow mais lento
```

Exemplo:
- Flow 1 = 10 min
- Flow 2 = 2 min

👉 Job termina em **10 min**, não 12

## 13. Comportamento em caso de falha

Se qualquer Flow falhar:
- O Job termina com status `FAILED`

Mesmo que:
- Outros Flows tenham terminado com sucesso

📌 O status final do Job é consolidado entre todos os Flows.

## 14. Importante: falha NÃO interrompe automaticamente outros Flows

Se:
- Flow 2 falha em 2 minutos
- Flow 1 leva 10 minutos

👉 O Spring Batch **aguarda Flow 1 terminar**
👉 Só depois marca o Job como `FAILED`

📌 Threads paralelas **não são canceladas automaticamente** por padrão.

## 15. Caso de uso ideal

Steps Paralelos são ideais quando:
- Há **pipelines independentes**
- Não existe dependência de ordem
- Não compartilham estado
- O ganho vem de **execução concorrente de blocos diferentes**

## 16. Quando NÃO usar Steps Paralelos?

❌ Quando os Steps:
- dependem do resultado um do outro
- compartilham variáveis ou contexto
- precisam de ordenação
- processam o mesmo conjunto de dados

👉 Nesse caso, prefira:
- Multithread Step (chunks paralelos)
- Partitioning

## 17. Distinção final (modelo mental)

### Steps Paralelos
- Várias linhas de produção diferentes rodando ao mesmo tempo.

### Multithread Step
- Vários operários trabalhando na mesma linha de produção.

## 18. 📌 Conclusão

Mesmo Step pesado? → **Multithread Step**

Steps independentes? → **Parallel Steps**

# Processamento Assíncrono no Spring Batch (Async Processing)

## 1. Objetivo da técnica

O Processamento Assíncrono é usado quando o gargalo do Step está no processamento (`ItemProcessor`), especialmente em cenários I/O-bound, como:
- Chamadas REST externas
- Integrações com APIs
- Enriquecimento remoto de dados
- Operações com latência alta

📌 A meta é **aumentar o throughput liberando a thread principal** enquanto tarefas lentas são executadas em paralelo.

## 2. Quando faz sentido usar?

Use Async Processing quando:
- O ItemProcessor é **mais lento que Reader e Writer**
- Existe **latência externa** (REST, RPC, serviços)
- O custo está em **esperar I/O**, não CPU
- Processar item a item de forma sequencial gera gargalo

❌ Não é ideal quando:
- O gargalo está no banco de dados
- O Writer é o ponto mais lento
- Os itens têm tempo de processamento muito desigual (causa bloqueios longos)

## 3. Componentes principais

Para implementar:
- `AsyncItemProcessor`
- `AsyncItemWriter`
- `TaskExecutor` (pool de threads)

📌 O processamento deixa de ser síncrono e passa a ser delegado para um pool de threads.

## 4. Modelo mental: delegação (não uma thread por item)

❌ **Não** é criada uma thread nova para cada item (isso seria caro demais)

✅ Em vez disso:
- Os itens são enviados para um pool de threads
- Cada thread processa itens conforme disponibilidade

📌 O Async Processing usa reuso de threads, não criação indiscriminada.

## 5. Fluxo interno do Step com Async Processing

### Thread Principal (Master)
1. Lê o item (Reader)
2. Envia o item para o `AsyncItemProcessor`
3. Recebe imediatamente um **Future** (promessa)
4. Continua lendo novos itens sem esperar o resultado

### Threads do Pool
- Executam o processamento real
- Fazem chamadas REST (por exemplo)
- Produzem o resultado futuro

📌 A Thread Principal **não bloqueia esperando o Processor.**

## 6. Mudança importante no contrato do Processor

### Processor normal:

```
Item → ItemProcessado
```

### AsyncItemProcessor:

```
Item → Future<ItemProcessado>
```

Ou seja:
- O retorno não é o dado pronto
- É uma promessa de que ele estará pronto no futuro

## 7. Por que o AsyncItemWriter é obrigatório?

O `ItemWriter` comum:
- Espera objetos reais
- Não sabe lidar com `Future<T>`
- Tentaria gravar a promessa → erro (ClassCastException)

O `AsyncItemWriter` resolve isso

Funções:
- Receber `Future<T>`
- Esperar o processamento terminar
- Extrair o valor real
- Delegar a gravação para um Writer tradicional

📌 Ele é um coordenador, não um Writer real.

## 8. Papel do .delegate() no AsyncItemWriter

O AsyncItemWriter **não grava nada sozinho.**

Ele:
1. Aguarda os Futures
2. Extrai os objetos reais
3. **Encaminha para um Writer real**, como:
    - `JdbcBatchItemWriter`
    - `FlatFileItemWriter`
    - qualquer writer padrão

📌 Isso permite reutilizar Writers existentes sem modificá-los.

## 9. Ordem de escrita e integridade do chunk

Mesmo que:
- Item 3 finalize antes do Item 1

👉 O `AsyncItemWriter`:
- aguarda todos os itens do chunk
- só escreve quando todos os Futures estão resolvidos

📌 O Spring Batch preserva a integridade transacional do chunk.

### Motivo:
- Se der erro → rollback do lote inteiro
- Nenhum item parcial é commitado

## 10. Ponto crítico: sincronização no Writer

O `AsyncItemWriter` funciona como uma barreira **(synchronization point)**:
- Junta todos os resultados do chunk
- Espera o mais lento terminar
- Só então libera a escrita

📌 O chunk avança na velocidade do item mais lento.

## 11. Efeito colateral: gargalo no item mais lento (Comboio)

### Cenário:
- Chunk = 1000 itens
- Item 1 = 10 segundos
- Itens 2–1000 = 10 ms

Resultado:
- Writer **fica bloqueado esperando o Item 1**
- Todo o chunk fica parado

📌 O desempenho final é limitado pelo **pior caso do lote**.

## 12. Impacto na Thread Principal

Mesmo com processamento assíncrono:
- A Thread Principal continua sendo sequencial
- Ciclo continua:

    ```
    Read → Process → Write
    ```

Se o Writer está bloqueado:
- O Reader para
- O próximo chunk não é lido
- A thread principal fica bloqueada

📌 O processamento é paralelo, mas o controle do Step **não é totalmente paralelo.**

## 14. Benefício real da técnica

### Ganha:
- Paralelismo no processamento
- Melhor uso de tempo ocioso (latência externa)
- Redução do tempo total do Step

### Não resolve:
- Writer lento
- Banco lento
- Chunk travado por item extremamente lento

## 15. Quando Async Processing é ideal?

✔ Chamadas REST

✔ Integrações externas

✔ I/O remoto

✔ Enriquecimento de dados

✔ Processamento com latência variável

## 16. Quando NÃO é ideal?

❌ Writer é gargalo

❌ Processamento é CPU-bound pesado

❌ Itens têm latência muito desigual

❌ Chunk muito grande com risco de item lento bloquear tudo

# Particionamento

Para minimizar gargalos de operações de I/O (leitura e escrita de dados), o framework permite a utilização de Workers capazes de executar leitor, processador e escritor como Steps independentes. Essa técnica é chamada de **particionamento**, que pode ser **local** ou **remoto**.

Com o particionamento, o Step Master é dividido em **múltiplas partições**, e cada uma delas é processada em paralelo por um Step Worker.

Cada Worker é um Step completo e independente, contendo:

- ItemReader
- ItemProcessor
- ItemWriter

Por serem Steps independentes, cada partição mantém seu próprio estado no JobRepository, o que permite que o Job continue sendo **restartável.**

## 1. Funcionamento Interno

O particionamento é composto por três elementos principais:

### 1.1 Master Step

Responsável por:

- Dividir o trabalho em partições
- Delegar a execução aos Workers
- Controlar a agregação do resultado final

Ele não executa processamento de dados diretamente.

### 1.2 Partitioner

A lógica de divisão das partições é implementada no `Partitioner`.

O `Partitioner`:

- Recebe o `gridSize`
- Cria um `ExecutionContext` para cada partição
- Retorna um `Map<String, ExecutionContext>`

Cada `ExecutionContext` contém os parâmetros necessários para que o Worker processe apenas sua fatia dos dados.

Exemplo de cenários de particionamento:

- Intervalo de IDs (ex: id 1–1000, 1001–2000…)
- Faixas de datas
- Arquivos diferentes
- Segmentos de tabela

O `gridSize` informa ao framework **quantas partições serão criadas**, mas não necessariamente o número exato de threads — isso depende da configuração do executor.

### 1.3 PartitionHandler

O `PartitionHandler` é responsável por definir **como** os Workers serão executados.

Existem três implementações principais:

- `TaskExecutorPartitionHandler`

→ Executa os Workers localmente, na mesma JVM, utilizando um TaskExecutor.

- `MessageChannelPartitionHandler`

→ Executa Workers em JVMs remotas utilizando mensageria.

- `DeployerPartitionHandler`

→ Utiliza Spring Cloud para criar Workers sob demanda e escalar dinamicamente.

Quando falamos de **Particionamento Local**, estamos nos referindo ao uso do:

`TaskExecutorPartitionHandler`

Nesse caso:

- Todos os Workers rodam na mesma aplicação
- A paralelização ocorre via threads
- É necessário configurar um TaskExecutor (ex: ThreadPoolTaskExecutor)

## 2. Cuidados Importantes no Particionamento Local

### 2.1 Banco de Dados Compartilhado

No particionamento local, os metadados do Step Master e dos Workers são salvos no mesmo **JobRepository**.

Isso significa que:

- Todos os Steps devem utilizar o mesmo banco de dados
- O banco deve suportar concorrência adequada
- Índices nas tabelas do JobRepository são fundamentais para evitar gargalos

### 2.2 Definição correta do gridSize

O `gridSize` deve ser definido com cautela:

- Pouco grid → não aproveita paralelismo
- Muito grid → excesso de threads e contenção de recursos

#### Regra prática:

Para processamento CPU-bound:
- gridSize próximo ao número de núcleos

Para processamento I/O-bound:
- pode ser maior que o número de núcleos

Mas sempre considerando:

- Conexões disponíveis no pool
- Limite do banco
- Memória da JVM

### 2.3 Leitores e Escritores Thread-Safe

No particionamento:

- Cada Worker possui sua própria instância de Reader/Processor/Writer, porém, se houver recursos compartilhados, é necessário garantir thread-safety

Evite:

- Uso de variáveis estáticas
- Compartilhamento manual de estado

### 2.4 Estratégia de divisão coerente

O particionamento só faz sentido quando:

- É possível dividir o conjunto de dados em partes independentes
- Não há dependência entre registros
- Não há necessidade de ordenação global durante processamento

Se houver dependência entre registros, o particionamento **pode causar inconsistências.**

## 3. Quando faz sentido utilizar Particionamento Local?

### ✅ Processamento de grandes volumes de dados

- Milhões de registros em tabela
- Processamento de grandes arquivos CSV

### ✅ Atualizações por faixa de ID

- Atualizar status de pedidos em lote
- Reprocessar dados históricos

### ✅ Processamento por período

- Fechamento mensal dividido por intervalo de datas

### ✅ Sistemas com infraestrutura limitada

- Não há necessidade de distribuir para múltiplas máquinas
- A aplicação já roda em servidor com múltiplos núcleos
- Quer paralelismo simples sem arquitetura distribuída

## 4. Quando NÃO utilizar?

❌ Quando o volume é pequeno

❌ Quando há forte dependência entre registros

❌ Quando o gargalo está na CPU e não em I/O

❌ Quando o banco não suporta alto nível de concorrência

## 5. Comparação Rápida: Multithread Step vs Particionamento

| **Multithread Step**| **Particionamento**                |
| ------------------  | ---------------------------------- |
| Um único Step       | Master + múltiplos Workers         |
| Compartilha Reader  | Cada Worker tem seu próprio Reader |
| Mais simples        | Mais escalável                     |
| Menor isolamento    | Maior isolamento                   |

Particionamento é mais robusto quando o objetivo é **escalar processamento mantendo restartabilidade forte e isolamento por partição.**

## 6. Particionamento Remoto

No **Particionamento Remoto**, o Step Master (Manager) delega o processamento das partições para Workers que executam em **outras JVMs**, possivelmente em **outras máquinas.**

A comunicação entre Manager e Workers ocorre através de um **Message Broker**, que atua como middleware de mensageria.

Exemplos de brokers comumente utilizados:

- RabbitMQ
-ActiveMQ
- Kafka

Nesse modelo:

- O Manager envia as partições via mensagens
- Os Workers recebem a partição
- Executam o Step correspondente
- Retornam o resultado ao Manager

A execução deixa de ser baseada em threads locais e passa a ser baseada em **mensageria distribuída.**

## 6.1 Arquitetura do Particionamento Remoto

A arquitetura envolve:

### 6.1.1 Manager (Master)

Responsável por:

- Criar as partições via `Partitioner`
- Enviar cada `ExecutionContext` para o broker
- Aguardar a finalização das partições
- Consolidar o resultado final

O Manager **não processa dados diretamente.**

### 6.1.2 Workers

Cada Worker:

- Escuta mensagens do broker
- Recebe uma partição
- Executa Reader, Processor e Writer
- Atualiza seu estado no JobRepository
- Retorna o status ao Manager

Cada Worker é uma aplicação Spring Batch independente.

### 6.1.3 Message Broker

O Broker é responsável por:

- Garantir entrega das mensagens
- Controlar filas
- Distribuir carga entre Workers
- Permitir escalabilidade horizontal

Ele desacopla Manager e Workers.

## 6.2 Implementação

Para implementar o Particionamento Remoto é necessário:

- Customizar um `Partitioner` para gerar as partições e seus respectivos `ExecutionContext`
- Utilizar o `MessageChannelPartitionHandler` para integrar com o Message Broker
- Configurar canais de envio e recebimento de mensagens (Spring Integration)

Diferentemente do particionamento local, aqui não utilizamos `TaskExecutorPartitionHandler`.

## 6.3 Banco de Dados e Restart

Assim como no particionamento local:

- Manager e Workers devem utilizar o **mesmo JobRepository**
- Devem apontar para o **mesmo banco de dados**

Isso é essencial para:

- Garantir consistência de estado
- Permitir restart correto em caso de falha
- Evitar reprocessamento indevido

Sem banco compartilhado, a restartabilidade é comprometida.

## 6.4 Quando utilizar Particionamento Remoto?

### ✅ Quando uma única máquina não é suficiente

- Se CPU, memória ou conexões de I/O se tornarem gargalos.

### ✅ Quando o volume de dados é muito grande

- Processamento massivo noturno
- Reprocessamento histórico de anos de dados

### ✅ Quando é necessário escalar horizontalmente

- Adicionar mais Workers conforme necessidade.

### ✅ Quando o gargalo está em I/O distribuído

- Múltiplos bancos
- Integrações externas
- Escrita em sistemas distintos

## 6.5 Cuidados Importantes

### 6.5.1 Ordem de inicialização

Os Workers devem estar ativos antes do Manager iniciar o envio das partições.

Caso contrário:

- As mensagens podem ficar acumuladas
- Pode ocorrer timeout
- Pode haver falha de execução

### 6.5.2 Latência de rede

Particionamento remoto só compensa quando:

- A latência de rede é baixa
- O custo de comunicação é menor que o ganho de paralelismo

Caso contrário, o overhead de mensageria pode anular o benefício.

### 6.5.3 Idempotência

Como há comunicação distribuída:

- Mensagens podem ser reenviadas
- Workers podem falhar no meio do processamento

É altamente recomendado que o processamento seja **idempotente.**

### 6.5.4 Complexidade operacional

Particionamento remoto adiciona:

- Broker
- Monitoramento distribuído
- Deploy de múltiplas aplicações
- Observabilidade mais complexa

É uma solução arquitetural mais pesada.

Use apenas quando necessário.

## 7. Comparação: Particionamento Local vs Remoto

| Aspecto        | Local          | Remoto                  |
| -------------- | -------------- | ----------------------- |
| Execução       | Mesma JVM      | Múltiplas JVMs          |
| Comunicação    | Threads        | Mensageria              |
| Escalabilidade | Vertical       | Horizontal              |
| Complexidade   | Média          | Alta                    |
| Infraestrutura | Simples        | Broker + múltiplas apps |
| Indicado para  | Carga moderada | Carga massiva           |

## 8. Quando NÃO utilizar Particionamento Remoto?

❌ Quando uma única máquina já atende a demanda

❌ Quando o gargalo não é I/O

❌ Quando a infraestrutura não comporta mensageria

❌ Quando o custo operacional não se justifica

## 9. Conclusão

Particionamento remoto é uma decisão arquitetural, não apenas técnica. Se você escolher essa abordagem cedo demais, você aumenta a complexidade antes de ter necessidade real.

A ordem madura geralmente é:

1. Step simples
2. Multithread Step
3. Particionamento Local
4. Particionamento Remoto

Escale conforme a dor aparece.

# Remote Chunking

O Remote Chunking é uma técnica de escalabilidade voltada principalmente para cenários onde o **gargalo do Job está no processamento (CPU-bound)** e não na leitura ou escrita.

Sua arquitetura também utiliza o modelo **Manager + Workers**, porém sua divisão de responsabilidades é diferente do Particionamento Remoto.

Enquanto no particionamento remoto cada Worker executa um **Step completo**, no Remote Chunking:

- O **Manager** realiza a leitura
- Os **Workers** executam o processamento e a escrita
- Cada chunk é enviado remotamente para execução

## 1. Diferença Conceitual

### 🔹 Particionamento Remoto

- Divide o Step em múltiplas partições
- Cada Worker executa Reader + Processor + Writer
- Estratégia baseada em divisão de dados

### 🔹 Remote Chunking

- Não divide o Step
- O Reader permanece no Manager
- O processamento e escrita são delegados remotamente
- Estratégia baseada em distribuição de chunks

## 2. Arquitetura do Remote Chunking

### 2.1 Manager

Responsável por:

- Executar o ItemReader
- Agrupar itens em chunks
- Enviar os chunks via Message Broker
- Aguardar retorno de status

O Manager **não executa o processamento nem a escrita.**

### 2.2 Workers

Cada Worker:

- Recebe um chunk via mensagem
- Executa o ItemProcessor
- Executa o ItemWriter
- Retorna status de sucesso ou falha

Os Workers não possuem Reader.

### 2.3 Message Broker

Assim como no particionamento remoto, a comunicação ocorre via mensageria:

- RabbitMQ
- ActiveMQ
- Kafka

O broker é responsável por:

- Entrega das mensagens
- Distribuição de carga
- Garantia de persistência (quando configurado)

## Fluxo de Execução

1. Manager lê os itens
2. Forma um chunk
3. Envia o chunk para o broker
4. Worker consome o chunk
5. Worker processa e escreve
6. Worker envia resposta
7. Manager controla commit e progresso

**Importante:** O commit continua sendo baseado em chunk, mas a execução ocorre remotamente.

## 4. Implementação

Para implementar Remote Chunking é necessário:

- Configurar canais de saída (Manager → Broker)
- Configurar canais de entrada (Worker → Broker)
- Definir IntegrationFlows (Spring Integration)
- Configurar mecanismos de serialização dos itens

Diferente do particionamento:

- Não é necessário criar um `Partitioner`
- Não há `gridSize`
- Não há divisão por faixa de dados

Qualquer Worker disponível pode consumir qualquer chunk.

## 5. Quando utilizar Remote Chunking?

### ✅ Quando o gargalo é CPU (processamento pesado)


- Criptografia
- Cálculos complexos
- Enriquecimento com regras pesadas
- Transformações complexas de dados

### ✅ Quando a leitura é rápida, mas o processamento é lento

- Se o banco entrega dados rapidamente, mas o processamento consome CPU intensivamente.

### ✅ Quando é necessário escalar horizontalmente apenas o processamento

- Você mantém leitura centralizada e distribui somente a parte pesada.

## 6. Cuidados Importantes

### 6.1 Latência de Rede

Remote Chunking só faz sentido se:

- A latência for baixa
- O custo de serialização + envio for menor que o ganho de paralelismo

Caso contrário, pode ficar mais lento que processamento local.

## 6.2 Garantia de Entrega

O Spring Batch **não armazena o conteúdo do chunk nos metadados do JobRepository.**

Isso significa:

Se um Worker cair durante o processamento, o framework precisa reenviar o chunk.

Para isso, o broker deve estar configurado com:

- Persistência de mensagens
- Acknowledgment adequado
- Política de redelivery

Sem isso, você pode perder dados.

### 6.3 Tamanho do Chunk

Chunk muito pequeno:

- Excesso de mensagens
- Overhead de rede

Chunk muito grande:

- Alto consumo de memória
- Tempo alto de processamento antes de commit

É necessário balancear:

- Custo de comunicação
- Tempo de processamento
- Capacidade dos Workers

### 6.4 Serialização

Os itens precisam ser serializáveis.

Evite:

- Objetos pesados
- Payloads com muitos dados desnecessários
- Dependência de estado local

## 7. Restartabilidade

O controle de restart continua sendo feito pelo Manager.

Se houver falha:

- Chunks não confirmados podem ser reenviados
- Workers devem ser idempotentes
- Escritas devem tolerar reprocessamento

Boas práticas:

- Chaves únicas no banco
- Operações UPSERT
- Processamento sem efeitos colaterais externos

## 11. Quando NÃO usar Remote Chunking?

❌ Quando o gargalo é I/O

❌ Quando leitura já é lenta

❌ Quando infraestrutura de mensageria não está madura

❌ Quando o processamento é leve

## 12. Conclusão

Remote Chunking é uma solução cirúrgica. Ele resolve um problema muito específico: **CPU distribuída mantendo leitura centralizada.**

Se você usar sem necessidade, você:

- Aumenta complexidade
- Introduz rede no caminho crítico
- Cria pontos de falha distribuídos

Use quando a dor justificar.

# Comparação mental com outras técnicas de performance em processamento batch

| Técnica                                          | Escala o quê?            | Resolve qual gargalo?   | Complexidade | Infraestrutura extra             | Quando usar                                               | Quando evitar                                  |
| ------------------------------------------------ | ------------------------ | ----------------------- | ------------ | -------------------------------- | --------------------------------------------------------- | ---------------------------------------------- |
| **Ajuste de Chunk Size**                         | Throughput por transação | Overhead de commit      | ⭐ Baixa      | Não                              | Quando precisa melhorar performance sem mudar arquitetura | Quando gargalo é CPU ou I/O pesado             |
| **JdbcBatchItemWriter**                          | Escrita em banco         | I/O de escrita          | ⭐ Baixa      | Não                              | Quando INSERT/UPDATE é gargalo                            | Quando problema está no processamento          |
| **Paginação ao invés de Cursor**                 | Consumo de memória       | Uso excessivo de RAM    | ⭐ Baixa      | Não                              | Grandes volumes de dados                                  | Quando latência de múltiplas queries prejudica |
| **Multithread Step**                             | Execução de chunks       | I/O moderado            | ⭐⭐ Média     | Não                              | Paralelismo simples na mesma JVM                          | Quando precisa isolamento forte                |
| **Particionamento Local**                        | Step inteiro             | I/O alto                | ⭐⭐⭐ Média    | Não                              | Grande volume de dados em uma única máquina               | Quando há dependência entre registros          |
| **Particionamento Remoto**                       | Step inteiro distribuído | I/O massivo             | ⭐⭐⭐⭐ Alta    | Message Broker / múltiplas JVMs  | Quando uma máquina não suporta a carga                    | Quando rede é lenta ou instável                |
| **Remote Chunking**                              | Processamento (CPU)      | CPU-bound               | ⭐⭐⭐⭐ Alta    | Message Broker                   | Processamento pesado, leitura rápida                      | Quando gargalo é leitura ou escrita            |
| **Banco Tunado (Índices / Pool / Configuração)** | I/O banco                | Lock, lentidão SQL      | ⭐⭐ Média     | Ajuste de infraestrutura         | Sempre que houver alto volume                             | Quando problema é CPU                          |
| **Aumento de Hardware Vertical**                 | CPU + RAM                | Limitação física        | ⭐ Baixa      | Upgrade servidor                 | Quando precisa solução rápida                             | Quando arquitetura já é gargalo                |
| **Escala Horizontal (Clusterização)**            | Capacidade total         | Limite de máquina única | ⭐⭐⭐⭐ Alta    | Orquestração / Infra distribuída | Sistemas críticos e de alto volume                        | Quando custo não compensa                      |

## Leitura Estratégica da Tabela

Agora vem a parte importante.

Antes de aplicar qualquer técnica, você precisa responder:

### 1️⃣ Onde está o gargalo?

- CPU?
- Escrita em banco?
- Leitura?
- Rede?
- Memória?
= Lock de tabela?

O erro clássico é aplicar paralelismo onde o problema é SQL mal otimizado.

### 2️⃣ O problema é arquitetural ou de configuração?

Exemplos:

- Índice faltando → não é problema de Spring Batch
- Pool JDBC mal dimensionado → não é problema de chunk
- Banco subdimensionado → não é problema de paralelismo

Sempre otimize na seguinte ordem:

1. SQL
2. Índices
3. Pool de conexões
4. Chunk size
5. Multithread
6. Particionamento
7. Distribuição remota

Subir para arquitetura distribuída cedo demais é erro de engenharia.