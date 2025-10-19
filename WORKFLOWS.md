# Documentação dos Workflows

Este documento explica em detalhes cada um dos workflows de exemplo incluídos neste repositório.

## 📁 Estrutura dos Workflows

```
.github/
└── workflows/
    ├── basic-pipeline.yml      # Conceitos básicos
    ├── linux-features.yml      # Recursos do Linux
    └── ci-cd-example.yml       # CI/CD completo
```

## 🚀 basic-pipeline.yml

### Propósito
Introduzir conceitos fundamentais do GitHub Actions de forma simples e clara.

### Jobs Incluídos

#### 1. `hello-world`
- **Objetivo**: Primeiro contato com GitHub Actions
- **O que demonstra**:
  - Execução de comandos simples
  - Uso de contextos (`${{ }}`)
  - Acesso a variáveis do GitHub
- **Comandos executados**:
  ```bash
  whoami          # Mostra o usuário atual
  pwd             # Mostra o diretório atual
  uname -a        # Informações do sistema
  df -h           # Espaço em disco
  ```

#### 2. `checkout-example`
- **Objetivo**: Mostrar como acessar o código do repositório
- **O que demonstra**:
  - Action `checkout` para baixar código
  - Navegação em arquivos
  - Leitura de conteúdo
- **Action utilizada**: `actions/checkout@v4`

#### 3. `environment-variables`
- **Objetivo**: Trabalhar com variáveis de ambiente
- **O que demonstra**:
  - Variáveis globais do job
  - Variáveis de step
  - Criação de variáveis personalizadas
  - Uso de `$GITHUB_ENV`

#### 4. `depends-on-others`
- **Objetivo**: Demonstrar dependências entre jobs
- **O que demonstra**:
  - Palavra-chave `needs`
  - Execução sequencial de jobs
  - Coordenação de workflows

### Quando é Acionado
- Push para branches `main` ou `master`
- Pull requests para `main` ou `master`
- Manualmente via `workflow_dispatch`

---

## 🐧 linux-features.yml

### Propósito
Explorar recursos específicos e capacidades do runner Linux (Ubuntu).

### Jobs Incluídos

#### 1. `system-info`
- **Objetivo**: Informações detalhadas do sistema
- **O que mostra**:
  - Distribuição Linux (`/etc/os-release`)
  - Versão do kernel
  - Recursos de hardware (CPU, RAM, disco)
- **Comandos chave**:
  ```bash
  cat /etc/os-release  # Informações da distribuição
  free -h              # Memória disponível
  lscpu                # Informações da CPU
  ```

#### 2. `package-management`
- **Objetivo**: Instalar pacotes do sistema
- **O que demonstra**:
  - Uso do `apt-get`
  - Atualização de pacotes
  - Instalação de ferramentas
- **Comandos utilizados**:
  ```bash
  sudo apt-get update
  sudo apt-get install -y curl wget jq
  ```

#### 3. `docker-examples`
- **Objetivo**: Trabalhar com Docker
- **O que demonstra**:
  - Docker pré-instalado no runner
  - Execução de containers
  - Uso de imagens Docker
- **Exemplos práticos**:
  ```bash
  docker run --rm hello-world
  docker run --rm python:3.11-slim python -c "print('Hello!')"
  ```

#### 4. `bash-scripting`
- **Objetivo**: Scripts bash avançados
- **O que demonstra**:
  - Funções em bash
  - Loops e condicionais
  - Arrays
  - Processamento de arquivos

#### 5. `preinstalled-tools`
- **Objetivo**: Mostrar ferramentas disponíveis
- **Ferramentas verificadas**:
  - Git, Vim
  - Python, Node.js, Go, Java
  - pip, npm, gem
  - make, cmake, gcc

#### 6. `permissions-security`
- **Objetivo**: Entender permissões
- **O que demonstra**:
  - Usuário `runner`
  - Uso de `sudo` sem senha
  - Manipulação de permissões de arquivos

#### 7. `networking`
- **Objetivo**: Capacidades de rede
- **O que demonstra**:
  - Interfaces de rede
  - Configuração DNS
  - Requisições HTTP com `curl`

### Quando é Acionado
- Push para branches `main` ou `master`
- Manualmente via `workflow_dispatch`

---

## 🔄 ci-cd-example.yml

### Propósito
Demonstrar padrões reais de CI/CD usados em projetos de produção.

### Jobs Incluídos

#### 1. `lint`
- **Objetivo**: Verificação de qualidade de código
- **O que verifica**:
  - Sintaxe de arquivos YAML
  - Arquivos Markdown
- **Importância**: Primeira linha de defesa contra erros

#### 2. `build`
- **Objetivo**: Compilar o projeto
- **O que demonstra**:
  - **Matrix Strategy**: Testa múltiplas versões (Node.js 16, 18, 20)
  - **Artifacts**: Salva resultado do build
  - **Setup Actions**: Configura ambiente
- **Actions utilizadas**:
  - `actions/setup-node@v4`
  - `actions/upload-artifact@v3`

#### 3. `test`
- **Objetivo**: Executar testes
- **O que demonstra**:
  - Testes unitários
  - Testes de integração
  - Relatórios de cobertura
  - Dependências (`needs: lint`)

#### 4. `security`
- **Objetivo**: Análise de segurança
- **O que verifica**:
  - Vulnerabilidades em dependências
  - Secrets expostos no código
- **Importância**: Segurança antes do deploy

#### 5. `deploy-staging`
- **Objetivo**: Deploy para ambiente de testes
- **O que demonstra**:
  - **Environments**: Ambiente "staging"
  - **Condicionais**: `if: github.ref == 'refs/heads/develop'`
  - Download de artifacts
  - Health checks
- **Quando executa**: Apenas no branch `develop`

#### 6. `deploy-production`
- **Objetivo**: Deploy para produção
- **O que demonstra**:
  - Deploy condicional
  - Ambiente protegido
  - Notificações
- **Quando executa**: Apenas em `main` ou `master`

#### 7. `cache-example`
- **Objetivo**: Otimização com cache
- **O que demonstra**:
  - Cache de dependências
  - Redução de tempo de build
  - Configuração de chaves de cache
- **Action utilizada**: `actions/cache@v3`

#### 8. `notify`
- **Objetivo**: Resumo do workflow
- **O que demonstra**:
  - Verificação de status de outros jobs
  - Execução condicional (`if: always()`)
  - Acesso a resultados (`needs.*.result`)

### Quando é Acionado
- Push para `main`, `master` ou `develop`
- Pull requests para `main` ou `master`
- Manualmente via `workflow_dispatch`

### Permissões
```yaml
permissions:
  contents: read  # Mínimo necessário
```

---

## 🎯 Conceitos Importantes

### Matrix Strategy
Permite executar o mesmo job com diferentes configurações:
```yaml
strategy:
  matrix:
    node-version: [16, 18, 20]
    os: [ubuntu-latest, windows-latest]
```

### Artifacts
Arquivos gerados durante o workflow que podem ser compartilhados entre jobs:
```yaml
- uses: actions/upload-artifact@v3
  with:
    name: meu-artifact
    path: dist/
```

### Cache
Armazena dependências para acelerar workflows futuros:
```yaml
- uses: actions/cache@v3
  with:
    path: ~/.npm
    key: ${{ runner.os }}-node-${{ hashFiles('**/package-lock.json') }}
```

### Environments
Permitem configurar proteções e segredos específicos:
```yaml
environment:
  name: production
  url: https://exemplo.com
```

### Dependências entre Jobs
Jobs podem depender de outros usando `needs`:
```yaml
deploy:
  needs: [build, test]  # Só executa se build e test passarem
```

---

## 📊 Visualizando os Workflows

### No GitHub
1. Vá até a aba **Actions** do repositório
2. Selecione um workflow na barra lateral
3. Clique em uma execução para ver detalhes
4. Expanda jobs e steps para ver logs

### Logs e Debug
Para logs mais detalhados, adicione:
```yaml
- name: Debug
  run: echo "::debug::Minha mensagem de debug"
```

Ou habilite debug logging nas configurações do repositório.

---

## 🔧 Customizando os Workflows

### Adicionar Notificações
```yaml
- name: Notificar Slack
  uses: slackapi/slack-github-action@v1
  with:
    webhook-url: ${{ secrets.SLACK_WEBHOOK }}
```

### Adicionar Testes Reais
Substitua os comandos simulados por comandos reais:
```yaml
- run: npm ci
- run: npm test
- run: npm run build
```

### Adicionar Deploy Real
Configure secrets e use tools de deploy:
```yaml
- name: Deploy to Vercel
  uses: amondnet/vercel-action@v20
  with:
    vercel-token: ${{ secrets.VERCEL_TOKEN }}
```

---

## 📚 Recursos Adicionais

- [Sintaxe de Workflows](https://docs.github.com/en/actions/reference/workflow-syntax-for-github-actions)
- [Contextos e Expressões](https://docs.github.com/en/actions/reference/context-and-expression-syntax-for-github-actions)
- [Actions no Marketplace](https://github.com/marketplace?type=actions)
- [Eventos que Acionam Workflows](https://docs.github.com/en/actions/reference/events-that-trigger-workflows)

---

## 💡 Dicas Práticas

1. **Use cache**: Economize minutos de build
2. **Paralelização**: Execute jobs independentes em paralelo
3. **Fail-fast**: Configure `fail-fast: false` para testar todas as versões
4. **Timeouts**: Defina timeouts para evitar jobs travados
5. **Secrets**: Sempre use secrets para dados sensíveis
6. **Minimal permissions**: Use apenas as permissões necessárias

---

## 🐛 Troubleshooting

### Workflow não está executando?
- Verifique se o arquivo está em `.github/workflows/`
- Verifique a sintaxe YAML
- Confirme que os eventos de trigger estão corretos

### Job falhou?
- Leia os logs detalhadamente
- Verifique permissões
- Confirme que dependências estão instaladas
- Use debug logging

### Cache não está funcionando?
- Verifique a chave do cache
- Confirme que o path está correto
- Cache pode estar expirado (limite de 7 dias)
