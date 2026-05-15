# Linux

## O que é o Linux?

Linux é um **kernel** — o núcleo de um sistema operacional.

O kernel é a parte central do sistema, responsável por:

- Gerenciar memória
- Gerenciar processos
- Controlar dispositivos de hardware
- Gerenciar sistema de arquivos
- Fazer a ponte entre software e hardware

O Linux foi criado em **1991** por **Linus Torvalds** como projeto pessoal, inspirado no Unix.

---

## Arquitetura e Modularidade

O Linux foi projetado desde sua concepção com uma **arquitetura modular**:

- Existe um núcleo central (kernel)
- Componentes adicionais (drivers, módulos) podem ser carregados sob demanda
- Nem tudo precisa estar carregado na memória o tempo todo

Essa modularidade traz vantagens como:

- Melhor performance
- Melhor gerenciamento de recursos
- Maior flexibilidade e possibilidade de customização

> Embora seja classificado tecnicamente como **monolítico**, o kernel Linux suporta módulos carregáveis, o que oferece flexibilidade semelhante a arquiteturas mais modulares.

---

## Distribuições Linux

O Linux por si só é apenas o kernel. Para se tornar um sistema operacional completo e utilizável, são necessários:

- Shell
- Utilitários
- Gerenciador de pacotes
- Ambiente gráfico (opcional)
- Ferramentas administrativas
- Bibliotecas

Uma **distribuição Linux (distro)** é o conjunto de:

> kernel Linux + ferramentas + gerenciador de pacotes + aplicações organizadas para uso

### Distribuições principais

- Debian
- Red Hat
- Slackware
- Arch Linux
- Gentoo

### Distribuições derivadas

Muitas distros são baseadas em outras:

- **Ubuntu** → derivado do Debian
- **Linux Mint** → derivado do Debian/Ubuntu
- **Kali Linux** → derivado do Debian
- **Pop!_OS** → derivado do Ubuntu
- **CentOS** → derivado do Red Hat
- **Rocky Linux** → compatível com Red Hat
- **AlmaLinux** → compatível com Red Hat

Cada derivação pode adicionar ferramentas próprias, modificar o foco (desktop, servidor, segurança), incluir componentes proprietários ou alterar ciclos de atualização.

O diferencial de uma distro normalmente está em:

- Gerenciador de pacotes
- Filosofia de atualização (rolling release vs. estável)
- Público-alvo
- Suporte comercial (ex: Red Hat)

---

## Licenças de Software

Licenças definem direitos de uso, regras de modificação, formas de redistribuição e obrigações legais. No ecossistema Linux, são fundamentais para garantir liberdade e colaboração.

### Principais licenças

| Licença | Tipo | Características |
|---|---|---|
| **GPL v2 / GPL v3** | Copyleft | Modificações devem ser distribuídas sob GPL. O kernel Linux usa GPLv2 |
| **MIT** | Permissiva | Permite uso comercial e incorporação em software proprietário |
| **Apache** | Permissiva | Inclui proteção contra patentes |
| **BSD** | Muito permissiva | Redistribuição com poucas restrições |
| **Creative Commons** | Variada | Voltada a conteúdo (textos, imagens), não a software |

- A **GPL** é mais protetiva da liberdade do código
- **MIT**, **BSD** e **Apache** são mais flexíveis

---

## GNU e Software Livre

### O que é GNU?

GNU é um projeto iniciado por **Richard Stallman em 1983** com o objetivo de criar um sistema operacional totalmente livre, compatível com Unix.

**GNU** significa: *GNU's Not Unix*

O projeto desenvolveu ferramentas essenciais como:

- Compiladores (GCC)
- Shell (Bash)
- Coreutils
- Bibliotecas (glibc)

### As 4 Liberdades do Software Livre

Segundo a **Free Software Foundation (FSF)**:

1. Liberdade de executar o programa como quiser
2. Liberdade de estudar o código-fonte
3. Liberdade de redistribuir
4. Liberdade de modificar e distribuir versões modificadas

### GNU + Linux

Quando o kernel Linux surgiu, ele foi combinado com as ferramentas do projeto GNU. Por isso, tecnicamente, o sistema completo é chamado por muitos de **GNU/Linux** — sem as ferramentas GNU, o kernel sozinho não seria suficiente para o usuário final.

---

## DFSG — Debian Free Software Guidelines

**DFSG** são diretrizes criadas pelo projeto **Debian** para definir o que pode ser considerado software livre dentro da distribuição.

Alguns princípios:

- Livre redistribuição
- Código-fonte disponível
- Permitir modificações
- Não discriminar pessoas ou áreas de atuação
- Não depender de outra licença específica

> As diretrizes DFSG influenciaram inclusive a definição de **Open Source**.

---

## BSD

**BSD** (Berkeley Software Distribution) é um sistema operacional Unix-like derivado do Unix original da AT&T.

Diferente do Linux:

- BSD já é um **sistema operacional completo** (não apenas o kernel)
- Inclui kernel + ferramentas base

Exemplos:

- FreeBSD
- OpenBSD
- NetBSD

Muitos conceitos e ferramentas utilizados no Linux possuem influência ou origem no ecossistema BSD.

---

## Resumo

| Componente | O que é |
|---|---|
| **Linux** | Kernel |
| **GNU** | Ferramentas e utilitários |
| **Distribuição** | Kernel + GNU + ferramentas + gerenciador de pacotes |
| **Licenças** | Definem liberdade e regras de uso |
| **DFSG** | Critérios de software livre do Debian |
| **BSD** | Sistema Unix-like completo |