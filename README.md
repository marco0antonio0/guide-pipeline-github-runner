# Guia de Pipeline do GitHub Runner no Linux

Este repositório é um guia completo para entender e implementar pipelines do GitHub Actions usando runners Linux.

## 📚 Documentação

- **[QUICK_START.md](QUICK_START.md)** - Guia rápido para começar
- **[WORKFLOWS.md](WORKFLOWS.md)** - Documentação detalhada dos workflows de exemplo
- **[ARCHITECTURE.md](ARCHITECTURE.md)** - Arquitetura e diagramas do GitHub Actions

## 📋 Índice

- [O que é GitHub Actions?](#o-que-é-github-actions)
- [O que é um GitHub Runner?](#o-que-é-um-github-runner)
- [Estrutura de uma Pipeline](#estrutura-de-uma-pipeline)
- [Componentes Principais](#componentes-principais)
- [Exemplos Práticos](#exemplos-práticos)
- [Boas Práticas](#boas-práticas)

## 🚀 O que é GitHub Actions?

GitHub Actions é uma plataforma de integração contínua e entrega contínua (CI/CD) que permite automatizar, personalizar e executar seus fluxos de trabalho de desenvolvimento de software diretamente no seu repositório GitHub.

### Principais Características:

- **Automação**: Execute testes, builds e deployments automaticamente
- **Integração nativa**: Totalmente integrado com o GitHub
- **Flexibilidade**: Suporte para múltiplas linguagens e plataformas
- **Gratuito**: Minutes gratuitos para repositórios públicos

## 🖥️ O que é um GitHub Runner?

Um GitHub Runner é um servidor que executa seus workflows quando são acionados. O GitHub fornece runners hospedados com Ubuntu Linux, Windows e macOS.

### Tipos de Runners:

1. **GitHub-hosted runners** (Fornecidos pelo GitHub)
   - Ubuntu (Linux)
   - Windows Server
   - macOS

2. **Self-hosted runners** (Auto-hospedados)
   - Configurados em sua própria infraestrutura

### Runners Linux:

Os runners Linux são baseados em Ubuntu e vêm pré-instalados com:
- Ferramentas de desenvolvimento (git, docker, etc.)
- Linguagens de programação (Node.js, Python, Java, etc.)
- Ferramentas de build e teste
- Utilitários do sistema

## 🏗️ Estrutura de uma Pipeline

Uma pipeline do GitHub Actions é definida em arquivos YAML dentro do diretório `.github/workflows/`.

### Estrutura Básica:

```yaml
name: Nome do Workflow
on: [eventos que acionam o workflow]
jobs:
  nome-do-job:
    runs-on: ubuntu-latest
    steps:
      - name: Nome do passo
        run: comando a ser executado
```

## 🔧 Componentes Principais

### 1. **Workflow**
Processo automatizado configurável que executa um ou mais jobs.

### 2. **Events (Eventos)**
Atividades específicas que acionam um workflow:
- `push`: Quando código é enviado ao repositório
- `pull_request`: Quando um PR é aberto/atualizado
- `schedule`: Em horários programados
- `workflow_dispatch`: Acionamento manual

### 3. **Jobs**
Conjunto de steps que executam no mesmo runner.

### 4. **Steps**
Tarefas individuais que executam comandos ou actions.

### 5. **Actions**
Aplicações reutilizáveis que executam tarefas específicas.

### 6. **Runner**
Servidor que executa os workflows.

## 💡 Exemplos Práticos

Este repositório contém exemplos práticos de pipelines:

### 1. Pipeline Básica (`basic-pipeline.yml`)
Demonstra conceitos fundamentais:
- Checkout de código
- Execução de comandos simples
- Múltiplos jobs

### 2. Pipeline Linux (`linux-features.yml`)
Mostra recursos específicos do Linux:
- Comandos bash
- Instalação de pacotes
- Variáveis de ambiente
- Uso do Docker

### 3. Pipeline CI/CD (`ci-cd-example.yml`)
Padrões comuns de CI/CD:
- Build e teste
- Linting
- Matrix strategy (múltiplas versões)
- Artifacts e caching

## 📚 Boas Práticas

### 1. **Use Caching**
```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### 2. **Defina Timeouts**
```yaml
jobs:
  build:
    timeout-minutes: 10
```

### 3. **Use Secrets para Dados Sensíveis**
```yaml
env:
  API_KEY: ${{ secrets.API_KEY }}
```

### 4. **Minimize o Escopo de Permissões**
```yaml
permissions:
  contents: read
```

### 5. **Use Matriz de Testes**
```yaml
strategy:
  matrix:
    node-version: [14, 16, 18]
```

## 🔍 Como Funciona o Runner Linux?

### Ciclo de Vida de uma Pipeline:

1. **Acionamento**: Um evento aciona o workflow
2. **Alocação**: GitHub aloca um runner Linux (Ubuntu)
3. **Preparação**: Runner prepara o ambiente
4. **Checkout**: Código é baixado do repositório
5. **Execução**: Steps são executados sequencialmente
6. **Limpeza**: Runner limpa o ambiente
7. **Relatório**: Resultados são reportados ao GitHub

### Ambiente do Runner Linux:

```bash
# Sistema Operacional
Ubuntu 20.04 ou 22.04 LTS

# Usuário
runner (com sudo sem senha)

# Diretório de Trabalho
/home/runner/work/<repo-name>/<repo-name>

# Recursos
- 2-core CPU
- 7 GB RAM
- 14 GB SSD
```

## 📖 Recursos Adicionais

- [Documentação Oficial do GitHub Actions](https://docs.github.com/en/actions)
- [Marketplace de Actions](https://github.com/marketplace?type=actions)
- [Especificações dos Runners](https://docs.github.com/en/actions/using-github-hosted-runners/about-github-hosted-runners)

## 🤝 Contribuindo

Sinta-se à vontade para contribuir com exemplos adicionais ou melhorias na documentação!

## 📝 Licença

Este projeto é de código aberto e está disponível para fins educacionais.