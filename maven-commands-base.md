# 🪶 Guia de Comandos Maven no Terminal

> Referência rápida dos principais comandos Maven para uso no terminal integrado do VS Code.  
> Para abrir o terminal: `Ctrl + Shift + `` ` `

---

## 📌 Conceitos Rápidos

| Termo | Significado |
|---|---|
| `mvn` | Invoca o Maven instalado globalmente na máquina |
| `./mvnw` | Invoca o Maven Wrapper do projeto (não exige Maven instalado globalmente — preferível em times) |
| `lifecycle` | Sequência de fases que o Maven executa em ordem (validate → compile → test → package → install → deploy) |
| `plugin:goal` | Executa um objetivo específico de um plugin sem passar pelo lifecycle completo (ex: `dependency:tree`) |
| `-D` | Passa uma propriedade/parâmetro para o Maven ou para a JVM |
| `-P` | Ativa um perfil definido no `pom.xml` |

> **`mvn` vs `./mvnw`:** Prefira `./mvnw` em projetos Spring Boot — ele garante que todos usem a mesma versão do Maven definida no projeto, independentemente do que está instalado na máquina.

---

## 🔁 Lifecycle do Maven (resumo)

```
validate → compile → test → package → verify → install → deploy
```

Cada fase **inclui todas as anteriores**. Ao rodar `mvn install`, o Maven executa validate, compile, test, package e verify antes de instalar.

---

## 🧹 Limpeza

### `mvn clean`
Remove o diretório `target/`, eliminando todos os artefatos gerados em builds anteriores.

**Quando usar:** Antes de qualquer build relevante, especialmente ao trocar de branch, ao suspeitar de artefatos corrompidos ou de cache desatualizado.

```bash
mvn clean
```

---

## 🔨 Compilação

### `mvn compile`
Compila os fontes de `src/main/java`. Não compila testes.

**Quando usar:** Para checar erros de compilação rapidamente sem executar testes ou gerar o `.jar`.

```bash
mvn compile

# Combinado com clean para garantir build limpa:
mvn clean compile
```

### `mvn clean compile` ⭐
Limpa artefatos anteriores e recompila do zero. Retorna apenas erros de compilação sem executar testes.

**Quando usar:** Durante o desenvolvimento, para validar se o código compila corretamente após mudanças estruturais (novos métodos, alterações de assinatura, refatorações).

```bash
mvn clean compile
```

---

## 🧪 Testes

### `mvn test`
Compila e executa os testes unitários (classes em `src/test/java`).

**Quando usar:** Para rodar a suíte de testes sem gerar o artefato final.

```bash
mvn test

# Rodar apenas uma classe de teste específica:
mvn test -Dtest=NomeDaClasseTest

# Rodar apenas um método específico:
mvn test -Dtest=NomeDaClasseTest#nomeDoMetodo

# Pular a execução dos testes (compilação de testes ainda ocorre):
mvn test -DskipTests

# Pular compilação e execução dos testes:
mvn test -Dmaven.test.skip=true
```

---

## 📦 Empacotamento

### `mvn package`
Executa o lifecycle até o empacotamento, gerando o `.jar` ou `.war` na pasta `target/`.

**Quando usar:** Para gerar o artefato executável sem instalá-lo no repositório local.

```bash
mvn package

# Ignorando testes para empacotar mais rápido:
mvn package -DskipTests
```

---

## 🏗️ Build Completa

### `mvn clean install` ⭐
Limpa, compila, testa, empacota e instala o artefato no repositório local Maven (`~/.m2/repository`).

**Quando usar:**
- Build completa antes de subir código ou abrir Pull Request
- Quando o projeto é uma dependência de outro projeto local (o `install` disponibiliza o artefato para outros módulos via `~/.m2`)
- Em projetos multi-módulo, para garantir que todos os módulos estejam atualizados

```bash
mvn clean install

# Pulando testes quando necessário (não recomendado para PRs):
mvn clean install -DskipTests
```

---

## 🔍 Análise de Dependências

### `mvn dependency:tree` ⭐
Exibe a árvore completa de dependências do projeto, incluindo dependências transitivas (dependências das dependências).

**Quando usar:**
- Identificar conflitos de versão entre dependências (ex: duas libs trazendo versões diferentes do Jackson)
- Entender por que uma dependência específica está sendo incluída no classpath
- Auditar o que está sendo empacotado na aplicação

```bash
mvn dependency:tree

# Filtrar por uma dependência específica:
mvn dependency:tree -Dincludes=com.fasterxml.jackson.core

# Filtrar por groupId parcial:
mvn dependency:tree -Dincludes=org.springframework.*

# Exportar para arquivo:
mvn dependency:tree > dependencias.txt
```

### `mvn dependency:analyze`
Analisa as dependências do projeto e aponta:
- Dependências declaradas no `pom.xml` que não são usadas
- Dependências usadas no código mas não declaradas explicitamente (vêm como transitivas)

**Quando usar:** Para limpar o `pom.xml` e evitar dependências desnecessárias no build.

```bash
mvn dependency:analyze
```

### `mvn dependency:resolve`
Baixa e resolve todas as dependências declaradas, sem realizar build.

**Quando usar:** Para "pré-aquecer" o repositório local em ambientes CI ou após clonar um projeto novo.

```bash
mvn dependency:resolve
```

---

## 🚀 Execução com Spring Boot

### `mvn spring-boot:run` / `./mvnw spring-boot:run`
Inicia a aplicação Spring Boot diretamente pelo Maven, sem precisar gerar o `.jar` antes.

**Quando usar:** Durante o desenvolvimento local, para iniciar a aplicação rapidamente com hot-reload ou para testar configurações.

```bash
mvn spring-boot:run

# Com perfil ativo:
mvn spring-boot:run -Dspring-boot.run.profiles=dev
```

---

## 🌍 Variáveis de Ambiente com Spring Boot

Uma necessidade comum é passar credenciais ou configurações sensíveis via variável de ambiente em vez de hardcodá-las no `application.properties`. Abaixo os padrões para cada sistema operacional.

---

### 🪟 Windows (PowerShell)

No PowerShell, as variáveis de ambiente são definidas com `$env:NOME="valor"` **antes** de chamar o Maven, e persistem apenas durante a sessão do terminal.

```powershell
$env:JOB_NAME="meuJob"
$env:PROFILE="dev"
mvn spring-boot:run
```

**Com múltiplas variáveis de datasource (ex: aplicação multi-tenant):**
```powershell
$env:SPRING_DATASOURCE_URL="jdbc:postgresql://localhost:5432/meubanco"
$env:SPRING_DATASOURCE_USERNAME="root"
$env:SPRING_DATASOURCE_PASSWORD="senha"
$env:APP_DATASOURCE_USERNAME="root"
$env:APP_DATASOURCE_PASSWORD="senha"
$env:SPRING_PROFILES_ACTIVE="dev"
mvn spring-boot:run
```

> ⚠️ **Atenção:** No PowerShell, `$env:VARIAVEL` define a variável apenas para a sessão atual — ela não persiste ao fechar o terminal. Para torná-la permanente, use `[System.Environment]::SetEnvironmentVariable("NOME", "valor", "User")`.

---

### 🐧 Linux / macOS (Bash/Zsh)

No Linux e macOS, as variáveis são declaradas na mesma linha do comando usando `\` para quebra de linha, passando-as diretamente como prefixo do processo.

```bash
SPRING_DATASOURCE_USERNAME=root \
SPRING_DATASOURCE_PASSWORD=senha \
APP_DATASOURCE_USERNAME=root \
APP_DATASOURCE_PASSWORD=senha \
./mvnw spring-boot:run
```

**Com perfil e porta customizada:**
```bash
SPRING_PROFILES_ACTIVE=dev \
SERVER_PORT=8081 \
SPRING_DATASOURCE_URL=jdbc:postgresql://localhost:5432/meubanco \
SPRING_DATASOURCE_USERNAME=root \
SPRING_DATASOURCE_PASSWORD=senha \
./mvnw spring-boot:run
```

> 💡 **Dica:** Para não expor senhas no histórico do shell, prefira usar um arquivo `.env` com ferramentas como `dotenv` ou `direnv`, ou exporte as variáveis em um script `.sh` que não seja versionado no repositório (adicione ao `.gitignore`).

---

### 📋 Alternativa: `-Dspring-boot.run.jvmArguments`

Outra forma de passar configurações pontualmente sem definir variáveis de ambiente no shell:

```bash
./mvnw spring-boot:run \
  -Dspring-boot.run.jvmArguments="\
  -DSPRING_DATASOURCE_USERNAME=root \
  -DSPRING_DATASOURCE_PASSWORD=senha"
```

---

## ⚙️ Perfis Maven (`-P`)

Perfis permitem ativar configurações diferentes do `pom.xml` dependendo do ambiente (dev, homologação, produção).

```bash
# Ativar um perfil definido no pom.xml:
mvn clean install -Pdev

# Ativar múltiplos perfis:
mvn clean install -Pdev,integrationTests

# Spring Boot com perfil Maven + perfil Spring:
mvn spring-boot:run -Pdev -Dspring-boot.run.profiles=dev
```

---

## 🏢 Projetos Multi-Módulo

Em projetos com múltiplos módulos Maven, é possível controlar quais módulos são processados:

```bash
# Build de todos os módulos a partir da raiz:
mvn clean install

# Build de um módulo específico e suas dependências internas:
mvn clean install -pl modulo-servico -am

# Build ignorando um módulo específico:
mvn clean install -pl !modulo-legado

# Rodar em paralelo (mais rápido em projetos grandes):
mvn clean install -T 4
```

| Flag | Significado |
|---|---|
| `-pl` | Seleciona módulos específicos (*project list*) |
| `-am` | Também processa os módulos dos quais o selecionado depende (*also make*) |
| `-amd` | Também processa os módulos que dependem do selecionado (*also make dependents*) |
| `-T N` | Executa com N threads em paralelo |

---

## 🗂️ Referência Rápida

| Comando | Para que serve | Quando usar |
|---|---|---|
| `mvn clean` | Limpa a pasta `target/` | Antes de builds importantes ou ao trocar de branch |
| `mvn clean compile` | Limpa e compila sem rodar testes | Checar erros de compilação rapidamente |
| `mvn test` | Roda os testes unitários | Validar testes sem gerar artefato |
| `mvn package` | Gera o `.jar`/`.war` | Gerar artefato sem instalar no repositório local |
| `mvn clean install` | Build completa + instala no `~/.m2` | Build final, antes de PR, projetos com dependência local |
| `mvn clean install -DskipTests` | Build completa sem testes | Quando os testes já foram validados |
| `mvn dependency:tree` | Exibe árvore de dependências | Investigar conflitos e dependências transitivas |
| `mvn dependency:analyze` | Analisa uso das dependências | Limpar dependências desnecessárias do `pom.xml` |
| `mvn spring-boot:run` | Inicia aplicação Spring Boot | Desenvolvimento local sem gerar `.jar` |