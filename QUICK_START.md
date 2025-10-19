# Guia Rápido - Quick Start

## 🚀 Início Rápido

Este repositório contém um guia completo sobre GitHub Actions e pipelines com runners Linux.

### 📚 O que você vai encontrar aqui:

1. **README.md** - Guia completo em português explicando:
   - O que é GitHub Actions
   - O que é um GitHub Runner
   - Estrutura de uma pipeline
   - Componentes principais
   - Boas práticas

2. **WORKFLOWS.md** - Documentação detalhada dos exemplos:
   - Explicação de cada workflow
   - Conceitos importantes
   - Dicas de customização

3. **Workflows de exemplo** (`.github/workflows/`):
   - `basic-pipeline.yml` - Conceitos básicos
   - `linux-features.yml` - Recursos do Linux
   - `ci-cd-example.yml` - CI/CD completo

## 🎯 Como usar este guia

### Passo 1: Entenda os conceitos
Leia o [README.md](README.md) para entender os fundamentos do GitHub Actions.

### Passo 2: Veja os workflows em ação
1. Vá até a aba **Actions** deste repositório
2. Selecione um workflow na barra lateral
3. Clique em "Run workflow" para executar manualmente

### Passo 3: Explore os exemplos
Abra os arquivos em `.github/workflows/` e veja como são estruturados:

```bash
.github/workflows/
├── basic-pipeline.yml      # ⭐ Comece por aqui!
├── linux-features.yml      # Recursos avançados
└── ci-cd-example.yml       # Padrões reais de CI/CD
```

### Passo 4: Aprofunde-se
Leia o [WORKFLOWS.md](WORKFLOWS.md) para entender cada workflow em detalhes.

### Passo 5: Adapte para seu projeto
Copie e modifique os workflows para seu próprio projeto!

## 💡 Dicas para Iniciantes

1. **Comece simples**: Use `basic-pipeline.yml` como ponto de partida
2. **Execute manualmente**: Use `workflow_dispatch` para testar
3. **Leia os logs**: Os logs são sua melhor ferramenta de debug
4. **Itere**: Faça pequenas mudanças e teste frequentemente

## 🔍 Estrutura de um Workflow

```yaml
name: Meu Workflow         # Nome que aparece na UI
on: [push]                 # Quando executar
jobs:
  meu-job:
    runs-on: ubuntu-latest # Onde executar
    steps:
      - name: Meu step     # O que fazer
        run: echo "Hello!"
```

## 📖 Recursos Essenciais

- [README.md](README.md) - Guia principal
- [WORKFLOWS.md](WORKFLOWS.md) - Documentação dos exemplos
- [Documentação Oficial](https://docs.github.com/en/actions)
- [Marketplace de Actions](https://github.com/marketplace?type=actions)

## ❓ Perguntas Frequentes

### Como executar um workflow?
1. Vá até a aba **Actions**
2. Selecione o workflow
3. Clique em "Run workflow"

### Por que meu workflow não está executando?
- Verifique se o arquivo está em `.github/workflows/`
- Confirme que o evento de trigger está correto
- Verifique a sintaxe YAML

### Como ver os logs?
1. Vá até **Actions**
2. Clique na execução do workflow
3. Clique no job
4. Expanda os steps

### Posso usar isso em projetos privados?
Sim! GitHub Actions funciona em repositórios públicos e privados (com limites de minutos gratuitos).

## 🤝 Contribuindo

Sinta-se à vontade para:
- Abrir issues com dúvidas
- Sugerir melhorias
- Adicionar novos exemplos
- Corrigir erros

## 📝 Próximos Passos

1. ✅ Leu este guia rápido
2. ⬜ Ler o [README.md](README.md)
3. ⬜ Explorar um workflow de exemplo
4. ⬜ Executar um workflow manualmente
5. ⬜ Criar seu próprio workflow
6. ⬜ Ler o [WORKFLOWS.md](WORKFLOWS.md) para se aprofundar

---

**Pronto para começar?** Abra o [README.md](README.md) e comece sua jornada com GitHub Actions! 🚀
