# Arquitetura do GitHub Actions

## 📊 Visão Geral do Fluxo

```
┌─────────────────────────────────────────────────────────────┐
│                     GitHub Repository                        │
│  ┌─────────────────────────────────────────────────────┐   │
│  │              .github/workflows/*.yml                 │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ Evento (push, PR, etc.)
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   GitHub Actions Service                     │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  1. Lê o workflow YAML                              │   │
│  │  2. Valida sintaxe                                  │   │
│  │  3. Agenda jobs                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ Aloca Runner
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                  GitHub-hosted Runner                        │
│                    (Ubuntu Linux)                            │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Ambiente Limpo:                                    │   │
│  │  - Ubuntu 20.04 ou 22.04                            │   │
│  │  - 2 CPU cores                                      │   │
│  │  - 7 GB RAM                                         │   │
│  │  - 14 GB SSD                                        │   │
│  │  - Ferramentas pré-instaladas                      │   │
│  └─────────────────────────────────────────────────────┘   │
│                                                              │
│  Executa Steps:                                             │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  Step 1: Checkout do código                         │   │
│  │           ↓                                         │   │
│  │  Step 2: Instalar dependências                      │   │
│  │           ↓                                         │   │
│  │  Step 3: Executar testes                            │   │
│  │           ↓                                         │   │
│  │  Step 4: Build                                      │   │
│  │           ↓                                         │   │
│  │  Step 5: Deploy                                     │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
                           │
                           │ Relatório de Status
                           ▼
┌─────────────────────────────────────────────────────────────┐
│                   Resultados no GitHub                       │
│  ┌─────────────────────────────────────────────────────┐   │
│  │  ✅ Job 1: Success (2m 30s)                         │   │
│  │  ✅ Job 2: Success (1m 45s)                         │   │
│  │  ❌ Job 3: Failed (0m 15s)                          │   │
│  └─────────────────────────────────────────────────────┘   │
└─────────────────────────────────────────────────────────────┘
```

## 🔄 Ciclo de Vida de um Workflow

### 1. Acionamento (Trigger)
```
Evento → GitHub detecta → Workflow é acionado
```

**Eventos comuns:**
- `push` - Código enviado
- `pull_request` - PR aberto/atualizado
- `schedule` - Cron job
- `workflow_dispatch` - Manual

### 2. Preparação do Runner

```
┌─────────────────────┐
│  Runner Pool        │
│  ┌───┐ ┌───┐ ┌───┐ │
│  │ R1│ │ R2│ │ R3│ │
│  └───┘ └───┘ └───┘ │
└─────────────────────┘
         │
         ▼ Seleciona um runner disponível
┌─────────────────────┐
│  Runner Alocado     │
│  ┌─────────────┐    │
│  │ Ubuntu      │    │
│  │ + Tools     │    │
│  │ + Docker    │    │
│  └─────────────┘    │
└─────────────────────┘
```

### 3. Execução de Jobs

#### Jobs em Paralelo (Padrão)
```
Workflow
├── Job A ──────────► ✅ (2 min)
├── Job B ──────────► ✅ (3 min)
└── Job C ──────────► ✅ (2 min)

Total: ~3 min (tempo do job mais longo)
```

#### Jobs Sequenciais (com `needs`)
```
Workflow
├── Job A ──────────► ✅ (2 min)
     └──► Job B ─────► ✅ (3 min)
           └──► Job C ► ✅ (2 min)

Total: 7 min (soma de todos)
```

#### Jobs com Dependências Complexas
```
       Job A (Build)
         /    \
        /      \
   Job B      Job C
   (Test)     (Lint)
        \      /
         \    /
       Job D (Deploy)
         
Se A falha → B, C, D não executam
Se B ou C falha → D não executa
```

### 4. Steps dentro de um Job

```
Job: build
┌─────────────────────────────────────────┐
│ Step 1: Checkout                        │
│ ┌─────────────────────────────────────┐ │
│ │ uses: actions/checkout@v4           │ │
│ │ Status: ✅ (5s)                     │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Step 2: Setup Node                      │
│ ┌─────────────────────────────────────┐ │
│ │ uses: actions/setup-node@v4         │ │
│ │ Status: ✅ (10s)                    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Step 3: Install Dependencies            │
│ ┌─────────────────────────────────────┐ │
│ │ run: npm ci                         │ │
│ │ Status: ✅ (45s)                    │ │
│ └─────────────────────────────────────┘ │
│                                         │
│ Step 4: Run Tests                       │
│ ┌─────────────────────────────────────┐ │
│ │ run: npm test                       │ │
│ │ Status: ❌ (15s)                    │ │
│ └─────────────────────────────────────┘ │
└─────────────────────────────────────────┘
Status: ❌ Failed
```

## 🏗️ Anatomia de um Runner Linux

### Sistema de Arquivos
```
/
├── home/
│   └── runner/
│       ├── work/
│       │   └── <repo-name>/
│       │       └── <repo-name>/     ← Seu código aqui
│       ├── .npm/                     ← Cache npm
│       ├── .cache/                   ← Cache geral
│       └── .local/                   ← Instalações locais
├── usr/
│   ├── bin/                          ← Ferramentas do sistema
│   └── local/                        ← Software adicional
└── opt/                              ← Ferramentas pré-instaladas
    ├── hostedtoolcache/              ← Versões de linguagens
    └── pipx/                         ← Ferramentas Python
```

### Ferramentas Pré-instaladas

```
┌─────────────────────────────────────────────────────────┐
│                    Ubuntu Runner                         │
├─────────────────────────────────────────────────────────┤
│  Linguagens:                                            │
│  • Python (3.8, 3.9, 3.10, 3.11)                       │
│  • Node.js (16, 18, 20)                                │
│  • Java (8, 11, 17, 21)                                │
│  • Go (1.20, 1.21)                                     │
│  • PHP, Ruby, .NET                                     │
├─────────────────────────────────────────────────────────┤
│  Ferramentas de Dev:                                    │
│  • Git, Docker, Docker Compose                         │
│  • kubectl, helm                                       │
│  • terraform, ansible                                  │
├─────────────────────────────────────────────────────────┤
│  Build Tools:                                           │
│  • make, cmake, gcc, g++                               │
│  • maven, gradle                                       │
│  • npm, yarn, pip                                      │
├─────────────────────────────────────────────────────────┤
│  Databases (para testes):                               │
│  • PostgreSQL, MySQL                                   │
│  • MongoDB, Redis                                      │
└─────────────────────────────────────────────────────────┘
```

## 🔐 Segurança e Permissões

### Estrutura de Permissões
```
┌────────────────────────────────────────┐
│  Repository (Nível mais alto)          │
│  ├── Secrets                           │
│  ├── Environments                      │
│  │   ├── production                    │
│  │   │   └── Secrets específicos      │
│  │   └── staging                       │
│  │       └── Secrets específicos      │
│  └── Workflow Permissions              │
│      ├── Read                          │
│      ├── Write                         │
│      └── Admin                         │
└────────────────────────────────────────┘
```

### Uso de Secrets
```yaml
# No workflow
env:
  API_KEY: ${{ secrets.API_KEY }}
  DB_PASSWORD: ${{ secrets.DB_PASSWORD }}

# Secrets nunca aparecem em logs
# Sempre são mascarados: ***
```

## 📊 Matrix Strategy

### Execução em Múltiplas Configurações

```
Matrix: 
  os: [ubuntu-latest, windows-latest, macos-latest]
  node: [16, 18, 20]

Gera 9 jobs:
┌────────────────────────────────────────┐
│  ubuntu + node 16  ✅                  │
│  ubuntu + node 18  ✅                  │
│  ubuntu + node 20  ✅                  │
│  windows + node 16 ✅                  │
│  windows + node 18 ❌                  │
│  windows + node 20 ✅                  │
│  macos + node 16   ✅                  │
│  macos + node 18   ✅                  │
│  macos + node 20   ✅                  │
└────────────────────────────────────────┘
Total: 8 passaram, 1 falhou
```

## 🎯 Padrões de CI/CD

### Pipeline Básico
```
┌──────┐   ┌──────┐   ┌────────┐
│ Lint │──►│ Test │──►│ Deploy │
└──────┘   └──────┘   └────────┘
```

### Pipeline Completo
```
         ┌──────┐
         │ Lint │
         └──┬───┘
            │
    ┌───────┴───────┐
    ▼               ▼
┌────────┐     ┌──────────┐
│  Test  │     │ Security │
└───┬────┘     └────┬─────┘
    │               │
    └───────┬───────┘
            ▼
      ┌──────────┐
      │  Build   │
      └────┬─────┘
           │
    ┌──────┴──────┐
    ▼             ▼
┌─────────┐   ┌────────────┐
│ Staging │   │ Production │
└─────────┘   └────────────┘
```

### Pipeline com Ambientes
```
main/master branch
       │
       ▼
┌─────────────┐
│   Build     │
└──────┬──────┘
       │
       ▼
┌─────────────┐     Aguarda
│  Staging    │────► Aprovação
└──────┬──────┘      Manual
       │                │
       ▼                │
   [Testes]◄───────────┘
       │
       ▼
┌─────────────┐     Requer
│ Production  │────► Review
└─────────────┘
```

## 💾 Cache e Artifacts

### Cache (Reutilização entre runs)
```
Run 1:
├── Build dependencies (5 min)
└── Save to cache

Run 2:
├── Restore from cache (10s) ✅
└── Build dependencies skipped!
```

### Artifacts (Compartilhamento entre jobs)
```
Job A: Build
├── Compila código
├── Gera dist/
└── Upload artifact "build"

Job B: Test
├── Download artifact "build"
├── Executa testes
└── Upload artifact "coverage"

Job C: Deploy
├── Download artifact "build"
└── Deploy para servidor
```

## 🔍 Debugging

### Visualização de Logs
```
Workflow Run #123
├── 📊 Summary
├── 📝 Jobs
│   ├── 🟢 build (2m 30s)
│   │   ├── Set up job (5s)
│   │   ├── Checkout (8s)
│   │   ├── Setup Node (12s)
│   │   ├── Install (1m 45s)
│   │   ├── Build (15s)
│   │   └── Post actions (5s)
│   │
│   └── 🔴 test (45s)
│       ├── Set up job (5s)
│       ├── Checkout (8s)
│       └── Run tests (32s) ← FALHOU AQUI
│           └── Error: Test suite failed
└── 📦 Artifacts
    └── test-results
```

## 📈 Boas Práticas de Arquitetura

### 1. Separação de Concerns
```yaml
workflows/
├── ci.yml          # Build e testes
├── cd.yml          # Deploy
├── security.yml    # Verificações de segurança
└── schedule.yml    # Tarefas agendadas
```

### 2. Reuso com Composite Actions
```yaml
# .github/actions/setup/action.yml
- uses: actions/checkout@v4
- uses: actions/setup-node@v4
- run: npm ci

# Uso:
- uses: ./.github/actions/setup
```

### 3. DRY com Reusable Workflows
```yaml
# .github/workflows/reusable-build.yml
on:
  workflow_call:

# Chamada:
jobs:
  build:
    uses: ./.github/workflows/reusable-build.yml
```

---

Este documento fornece uma visão arquitetural completa do GitHub Actions e como os workflows funcionam no runner Linux.
