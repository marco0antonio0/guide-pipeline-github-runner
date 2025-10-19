<p align="center">
  <img src="/Gemini_Generated_Image_86ne9786ne9786ne.png" width="170" alt="Github action Logo IA">
</p>
<p align="center">
  <img src="https://img.shields.io/badge/github-%23121011.svg?style=for-the-badge&logo=github&logoColor=white" alt="GitHub"/>
  <img src="https://img.shields.io/badge/github%20actions-%232671E5.svg?style=for-the-badge&logo=githubactions&logoColor=white" alt="GitHub Actions"/>
  <img src="https://img.shields.io/badge/docker-%230db7ed.svg?style=for-the-badge&logo=docker&logoColor=white" alt="Docker"/>
  <img src="https://img.shields.io/badge/linux-FCC624?style=for-the-badge&logo=linux&logoColor=black" alt="Linux"/>
  <img src="https://img.shields.io/badge/ubuntu-E95420?style=for-the-badge&logo=ubuntu&logoColor=white" alt="Ubuntu"/>
</p>

# GitHub Self-Hosted Runner Guide

## 1️⃣ O que é um Self-Hosted Runner?

Um **runner** é uma máquina que executa jobs de GitHub Actions. Existem dois tipos:

1. **GitHub-hosted runners:** Runners gerenciados pelo GitHub. Cada job inicia em uma VM limpa.
2. **Self-hosted runners:** Runners que você gerencia no seu próprio servidor. Permitem:

   * Uso de recursos dedicados (CPU, memória, GPU, armazenamento).
   * Execução em diretórios customizados.
   * Manutenção de estado entre jobs (ex.: caches de Docker, compiladores, arquivos de build).

---

## 2️⃣ Vantagens do Self-Hosted Runner

* Controle total sobre o ambiente.
* Execução mais rápida se você tiver máquinas potentes.
* Possibilidade de instalar ferramentas que não existem nos runners padrão.
* Repositórios privados ou forks podem usar o mesmo runner.

---

## 3️⃣ Requisitos mínimos

* Linux x64 (Ubuntu, Debian, CentOS, etc.) ou Windows/macOS.
* Node.js incluído no pacote do runner.
* Conexão com GitHub e token de registro (Settings → Actions → Runners → New self-hosted runner).

---

## 4️⃣ Instalando e configurando o runner

### Passo 1: Baixar o runner

```bash
mkdir -p ~/actions-runner && cd ~/actions-runner
curl -O -L https://github.com/actions/runner/releases/download/v2.329.0/actions-runner-linux-x64-2.329.0.tar.gz
tar xzf ./actions-runner-linux-x64-2.329.0.tar.gz
```

### Passo 2: Registrar o runner no GitHub

```bash
./config.sh --url https://github.com/<ORGANIZATION>/<REPO> --token <TOKEN>
```

Durante a configuração:

* Defina o **nome do runner** (ex.: `server`).
* Defina labels adicionais (ex.: `runner-default`).
* Confirme a pasta de trabalho (`_work` por padrão).

### Passo 3: Instalar como serviço (início automático)

```bash
sudo ./svc.sh install
sudo ./svc.sh start
sudo systemctl enable actions.runner.<repo>.<nome>.service
sudo systemctl status actions.runner.<repo>.<nome>.service
```

### Passo 4: Verificar se está pronto

No GitHub: Settings → Actions → Runners → deve aparecer **Idle** com labels configuradas.

---

## 5️⃣ Configurando o diretório de trabalho customizado

```yaml
jobs:
  deploy:
    runs-on: runner-default
    env:
      GITHUB_WORKSPACE: /workspace/Portfolio-v2/<repos>
```

O runner usará este diretório como base.

---

## 6️⃣ Exemplo de pipeline (Deploy Docker)

### Variáveis importantes no workflow

* **$GITHUB_WORKSPACE**: Diretório raiz do workspace do GitHub Actions no runner, formado por `<diretório do runner> + <nome do projeto>`.
* **Diretório do projeto**: Nome da pasta do seu projeto dentro do workspace, geralmente coincide com o nome do repositório.

```yaml
# =================================================================================
# GitHub Actions Workflow: Deploy Projeto
#
# Variáveis importantes:
# 
#   $GITHUB_WORKSPACE
#     - Diretório raiz do workspace do GitHub Actions no runner.
#     - Formado por: <diretório do runner> + <nome do projeto>
#
#   Diretório do projeto
#     - Nome da pasta do seu projeto dentro do workspace.
#     - Geralmente coincide com o nome do repositório.
#
# =================================================================================

name: Deploy Projeto

on:
  push:
    branches:
      - main

jobs:
  deploy:
    runs-on: runner-default
    # env:
    #   GITHUB_WORKSPACE: /home/server/workspace/REPO
    steps:
      - name: Checkout do código
        uses: actions/checkout@v4
        with:
          clean: false

      - name: Build Docker Compose
        run: |
          echo "[$(date)] Iniciando build Docker Compose"
          cd $GITHUB_WORKSPACE
          timeout 1200 docker compose build --progress=plain || { echo "Build falhou ou timeout"; exit 1; }

      - name: Parar containers antigos
        run: |
          cd $GITHUB_WORKSPACE
          echo "[$(date)] Parando containers antigos..."
          docker compose down --remove-orphans

      - name: Iniciar containers
        run: |
          cd $GITHUB_WORKSPACE
          echo "[$(date)] Iniciando containers..."
          docker compose up -d || { echo "Falha ao iniciar containers"; exit 1; }

      - name: Limpeza do sistema Docker
        run: |
          echo "[$(date)] Limpando sistema Docker com retry..."
          MAX_RETRIES=3
          RETRY_COUNT=0
          CLEANUP_SUCCESS=false

          while [ $RETRY_COUNT -lt $MAX_RETRIES ] && [ "$CLEANUP_SUCCESS" = false ]; do
            echo "[$(date)] Tentativa de limpeza $(($RETRY_COUNT + 1))/$MAX_RETRIES..."
            if docker system prune -f > /dev/null 2>&1; then
              echo "[$(date)] Limpeza concluída com sucesso"
              CLEANUP_SUCCESS=true
            else
              echo "[$(date)] Falha, aguardando 10 segundos..."
              sleep 10
              RETRY_COUNT=$(($RETRY_COUNT + 1))
            fi
          done

          if [ "$CLEANUP_SUCCESS" = false ]; then
            echo "[$(date)] AVISO: Limpeza não concluída, mas pipeline continua..."
          fi

      - name: Status final dos containers
        run: |
          cd $GITHUB_WORKSPACE
          echo "[$(date)] Status final dos containers:"
          docker ps --format "table {{.Names}}\t{{.Status}}"

      - name: Deploy finalizado
        run: |
          echo "[$(date)] Deploy finalizado com sucesso!"

```
