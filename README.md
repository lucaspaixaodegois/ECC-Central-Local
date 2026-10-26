# ECC Central (Agent Harness Performance Optimization System)

O **ECC Central** atua como o sistema operativo e a caixa de ferramentas avançada para agentes de inteligência artificial (como Claude Code, Cursor, OpenCode, entre outros) na infraestrutura local da FAPTO. Em vez de operar como um assistente de conversação genérico, este ecossistema força a IA a adotar um ciclo rigoroso de engenharia de software: **Planear → Testar (TDD) → Implementar → Rever → Verificar → Lembrar → Melhorar**.

Este repositório contém a infraestrutura do **Cérebro Central**, desenhada para correr isoladamente no servidor principal e servir múltiplos projetos simultaneamente (Buscador, PlayBola, SIAF, etc.) sem duplicação de recursos.

## 🏗️ Arquitetura: Cérebro Central e Operários Locais

A infraestrutura do ECC divide-se em duas camadas:

* **Cérebro Central (Este Projeto):** Hospedado de forma centralizada no servidor (`10.0.0.63`). Mantém o contexto partilhado (Memory Vault), aloja os 71 subagentes especializados, gere as 311 skills de desenvolvimento e assegura a segurança (AgentShield).
* **Operários Locais (Satélites):** Contentores leves injetados no ficheiro `docker-compose.yml` de cada projeto individual (ex: Buscador). Estes contentores não possuem a inteligência; apenas acedem aos ficheiros locais do projeto e conectam-se a este Cérebro Central para processamento.

## ✨ Funcionalidades Principais

* **Memory Vault (Redis):** Gere o consumo de tokens e cria um cofre de memória centralizado na porta `4000`. Permite que diferentes IDEs ou ferramentas de IA partilhem o mesmo histórico, contexto e progresso, independentemente do projeto ativo.
* **Gestão de Subagentes (Core):** Injeta 71 personas específicas no ambiente de desenvolvimento (arquitetos de software, especialistas em Go/Python/Flutter, analistas de segurança) para delegar tarefas consoante o contexto.
* **Habilidades Prontas (Skills):** Disponibiliza 311 fluxos de trabalho sob demanda, como desenvolvimento orientado a testes (TDD), aplicação de padrões OWASP e design de APIs RESTful.
* **Hooks de Automação:** Executa validações em segundo plano (ex: correr *linters* ou testes unitários sempre que a IA modifica um ficheiro).
* **AgentShield:** Camada de segurança interna que audita os comandos gerados pela IA para prevenir riscos como injeção de *prompt* ou exposição acidental de credenciais.

## 🚀 Requisitos da Infraestrutura

* **Docker & Docker Compose**
* **Central de IA Ativa:** O servidor principal (`10.0.0.63`) deve ter o *gateway* FastAPI e o motor Ollama a correr e expostos na porta `11434`.

## 🛠️ Instalação e Configuração

**1. Levantar a Infraestrutura Central**
Na raiz do projeto `ECC_Central`, execute o comando para iniciar o Memory Vault (Redis) e o Core do ECC:

```bash
docker compose up -d

```

**2. Instalar o Ecossistema ECC**
Aceda ao terminal do contentor principal para instalar os pacotes, os agentes e as skills, apontando o processamento pesado exclusivamente para a RTX 4060 da Central de IA:

```bash
docker compose exec ecc_core bash

```

Dentro do contentor, execute:

```bash
npx ecc-universal@2.2.3 install --profile full --target ollama --endpoint http://10.0.0.63:11434

```

## 🔗 Integração com Projetos Satélites (Operários)

Para que um projeto secundário (ex: Siaf) utilize esta infraestrutura, **não duplique o ECC**. Adicione apenas este bloco operário ao `docker-compose.yml` do projeto em questão:

```yaml
  ecc-operario-local:
    image: ubuntu:22.04
    container_name: ecc_operario_nome_do_projeto
    volumes:
      - .:/app
    working_dir: /app
    environment:
      - OLLAMA_HOST=http://10.0.0.63:11434
      - VAULT_URI=redis://10.0.0.63:4000  # Conecta ao Memory Vault central
      - LANG=C.UTF-8
    command: >
      bash -c "apt-get update && apt-get install -y curl git build-essential python3 python3-pip &&
      curl -fsSL https://deb.nodesource.com/setup_20.x | bash - &&
      apt-get install -y nodejs &&
      tail -f /dev/null"

```

Em seguida, instale a versão mínima (apenas de ligação) dentro desse operário:

```bash
npx ecc-universal@2.2.3 install --profile minimal --target ollama --endpoint http://10.0.0.63:11434

```