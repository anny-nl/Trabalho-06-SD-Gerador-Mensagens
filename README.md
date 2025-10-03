# Trabalho 6 - Sistemas Distribuídos (análise de imagens)

Este repositório contém um exemplo didático de um sistema distribuído utilizando Java, Docker e RabbitMQ.
O projeto possui um gerador de mensagens e dois consumidores que processam imagens (rostos e brasões de times).

IMPORTANTE: o README foi atualizado para refletir a estrutura atual do repositório — referências a scripts ou arquivos inexistentes foram removidas.

## Serviços (definidos em `docker-compose.yml`)

- `rabbitmq` — RabbitMQ com painel de administração (imagem: `rabbitmq:3-management`).
- `gerador-mensagens` — serviço Java que publica mensagens no broker.
- `rosto` — consumidor Java para processamento de imagens de rostos (contém `model.h5`).
- `brasao` — consumidor Java para identificação de brasões (contém embeddings/labels e `model.h5`).

## Estrutura do repositório

Arquivos e pastas relevantes:

- `docker-compose.yml` — orquestração dos serviços.
- `gerador-mensagens/` — Dockerfile, `Main.java`, `pom.xml`, bases em `base-rosto/` e `base-brasao/`.
- `rosto/` — Dockerfile, `Main.java`, `pom.xml`, `model.h5`.
- `brasao/` — Dockerfile, `Main.java`, `pom.xml`, `futebol_embeddings.txt`, `futebol_labels.txt`, `model.h5`.
- `rabbitmq/` — `definitions.json` e `rabbitmq.conf` (opcional para pré-configuração).
- `datasets_IA.ipynb`, `requirements.txt` — material adicional/experimentação.

## Como executar

Pré-requisitos:

- Docker Engine
- Docker Compose (comando `docker compose`) ou `docker-compose`

Na raiz do projeto:

```bash
docker compose down
docker compose up --build -d
docker compose ps
```

Ver logs:

```bash
docker compose logs -f <service-name>
```

A interface do RabbitMQ fica em http://localhost:15672 (credenciais padrão definidas no `docker-compose.yml`: `admin`/`admin123`).

## Parar os serviços

```bash
docker compose down
```

Ou parar containers do projeto por rótulo (exemplo):

```bash
docker stop $(docker ps -q --filter "label=com.docker.compose.project=trabalho_6_sd")
```

## Observações

- Alguns arquivos grandes (modelos `.h5`, embeddings) já estão incluídos nas pastas dos consumidores.
- Não há scripts `start.sh`/`iniciar_sistema.bat` no repositório — iniciar via `docker compose` é a forma suportada.

## Próximos passos (opcionais)

- Posso adicionar exemplos de como publicar mensagens em Java no README.
- Posso criar scripts de conveniência (`start.sh`, `stop.sh`) se desejar.

Se quiser que eu faça alguma dessas melhorias, diga qual e eu atualizo.

# Sistema Distribuído de Análise de Imagens com IA REAL - Trabalho 6 SD

Sistema distribuído em Java com containers Docker, RabbitMQ e **IA real embutida** nos consumidores para processamento e análise visual de imagens usando computer vision.

## 🏗️ Arquitetura do Sistema

O sistema é composto por 4 containers principais:

```
┌─────────────────┐    ┌─────────────────┐    ┌─────────────────┐
│  Gerador de     │    │                 │    │ Consumidor      │
│  Mensagens      │───▶│    RabbitMQ     │───▶│ Análise de      │
│  (6 msgs/seg)   │    │                 │    │ Sentimento      │
└─────────────────┘    │  Topic Exchange │    │ (Smile ML)      │
                       │                 │    └─────────────────┘
                       │   face_queue    │
                       │   team_queue    │    ┌─────────────────┐
                       │                 │    │ Consumidor      │
                       │                 │───▶│ Identificação   │
                       └─────────────────┘    │ de Times        │
                                              │ (Smile ML)      │
                                              └─────────────────┘
```

## 🚀 Componentes

### 1. **Gerador de Mensagens**

- **Função**: Gera carga constante de mensagens (6 mensagens/segundo)
- **Tipos de Imagem**:
  - 60% rostos de pessoas (routing key: `face`)
  - 40% brasões de times de futebol (routing key: `team`)
- **Tecnologias**: Java 11, RabbitMQ Client, Jackson JSON

### 2. **RabbitMQ Broker**

- **Exchange**: Topic (`image_analysis_exchange`)
- **Filas**:
  - `face_queue` (rostos)
  - `team_queue` (times)
- **Interface Admin**: http://localhost:15672 (admin/admin123)
- **Configuração**: Pré-configurado com definições JSON

### 3. **Consumidor de Análise de Sentimento (IA REAL)**

- **Função**: Processa imagens de rostos com **análise visual real**
- **IA**: Algoritmos de computer vision nativos em Java (BufferedImage, AWT)
- **Análise Real**:
  - Brilho médio pixel-a-pixel
  - Saturação de cores
  - Contraste e histograma
  - Detecção de cores quentes (felicidade)
- **Tempo**: 2-4 segundos por mensagem
- **Saída**: FELIZ/TRISTE com características visuais detalhadas

### 4. **Consumidor de Identificação de Times (IA REAL)**

- **Função**: Identifica brasões através de **análise de cores dominantes**
- **IA**: Computer vision para extração de características reais
- **Análise Real**:
  - % pixels vermelhos, verdes, azuis
  - % pixels pretos e brancos
  - Complexidade visual (variação de cores)
  - Classificação por regras de cor
- **Tempo**: 3-5 segundos por mensagem
- **Base**: 8 times com regras de cores específicas
- **Saída**: Nome do time com score de correspondência visual

## 🛠️ Tecnologias Utilizadas

- **Java 11** - Linguagem de programação
- **Maven** - Gerenciamento de dependências
- **Docker & Docker Compose** - Containerização
- **RabbitMQ** - Message Broker
- **Java AWT/BufferedImage** - **Processamento real de imagem**
- **Computer Vision** - **Algoritmos nativos de análise visual**
- **Jackson** - Processamento JSON
- **Python + Pillow** - Gerador de imagens de teste

## 📋 Pré-requisitos

- Docker Desktop instalado
- Docker Compose disponível
- Portas 5672 e 15672 livres

## 🎯 Como Executar

### ⚠️ Pré-requisitos

1. **Inicie o Docker Desktop** antes de executar o sistema
2. Aguarde o Docker estar completamente carregado
3. Verifique se as portas 5672 e 15672 estão livres

# Subir e construir todos os serviços

docker compose up --build -d

# Parar e remover containers/recursos criados anteriormente pelo compose

docker compose down

# Verificar status

docker compose ps

# Ver logs (ex.: rabbitmq)

docker compose logs -f rabbitmq

````

Observações:

- A interface de administração do RabbitMQ ficará disponível em http://localhost:15672 (credenciais padrão no compose: `admin` / `admin123`).
- As portas mapeadas estão definidas no `docker-compose.yml` (por padrão 5672 para AMQP e 15672 para a interface de administração).

## Como parar os serviços

Você pode remover os serviços com:

```bash
docker compose down
````

Ou parar apenas os containers do projeto com:

```bash
docker stop $(docker ps -q --filter "label=com.docker.compose.project=trabalho_6_sd")
```

Caso queira parar manualmente por nome (nomes exibidos por `docker compose ps`):

```bash
docker stop gerador-mensagens rosto brasao rabbitmq
```

## Notas sobre o conteúdo dos serviços

- `rosto` e `brasao` incluem artefatos (por exemplo `model.h5`, `futebol_embeddings.txt`) utilizados durante a execução. Esses arquivos já estão presentes nas respectivas pastas.
- O gerador de mensagens é implementado em Java (`Main.java`) e utiliza RabbitMQ para publicar mensagens nas filas do broker.

## Arquivos de configuração importantes

- `docker-compose.yml` — orquestração dos serviços, volumes e rede.
- `rabbitmq/definitions.json` — definições opcionais para pré-configurar exchanges/filas no RabbitMQ.

## Solução de problemas rápida

- Se um serviço não inicia, verifique os logs:

```bash
docker compose logs -f <service-name>
```

- Se o RabbitMQ não aceitar conexões imediatamente, aguarde o healthcheck (alguns segundos) e verifique as variáveis de ambiente no `docker-compose.yml`.

## Contribuição e autores

Projeto mantido por Jonas. Use issues/pull requests para propor mudanças.

---

Se quiser, eu posso:

- Incluir um exemplo mínimo de como publicar uma mensagem (trecho de Java) no README.
- Adicionar scripts de inicialização (bash) para conveniência.
- Gerar um pequeno diagrama atualizado.

Diga o que prefere e eu atualizo.
Análise Visual Real - Brilho: 0.72, Saturação: 0.65, Contraste: 0.41, Cores quentes: 45%

[TIMES] Processando: logo_flamengo_1.jpg  
[TIMES] ⚽ Identificado: FLAMENGO - RJ (87% confiança) (4.1s)
Classificação visual por cores - Score: 0.89 (R:67%, V:12%, A:8%)

````

## 🔧 Configurações

### Variáveis de Ambiente

| Variável | Padrão | Descrição |
|----------|--------|-----------|
| RABBITMQ_HOST | localhost | Host do RabbitMQ |
| RABBITMQ_PORT | 5672 | Porta do RabbitMQ |
| RABBITMQ_USER | admin | Usuário do RabbitMQ |
| RABBITMQ_PASS | admin123 | Senha do RabbitMQ |

### Taxas de Processamento

- **Gerador**: 6 mensagens/segundo
- **Consumidor Sentimento**: 2-4 segundos/mensagem
- **Consumidor Times**: 3-5 segundos/mensagem

## 🐛 Solução de Problemas

### Container não inicia
```bash
# Verificar logs
docker-compose logs [nome-do-serviço]

# Reconstruir do zero
docker-compose down
docker system prune -f
docker-compose up --build
````

### RabbitMQ não conecta

- Aguardar 30-60 segundos após o start (healthcheck)
- Verificar se a porta 5672 está livre
- Checar logs do container rabbitmq

### Biblioteca Smile não encontrada

- O Maven baixa automaticamente durante o build
- Em caso de erro, limpar cache: `docker system prune -f`

## 📁 Estrutura do Projeto

```
sistema-carga-ia/
├── docker-compose.yml              # Orquestração dos containers
├── start.bat / start.sh           # Scripts de inicialização
├── README.md                      # Documentação
├── gerador-mensagens/             # Serviço gerador
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/main/java/com/exemplo/
├── rosto/         # IA análise de sentimento
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/main/java/com/exemplo/
├── brasao/              # IA identificação de times
│   ├── Dockerfile
│   ├── pom.xml
│   └── src/main/java/com/exemplo/
├── rabbitmq/                # Configurações RabbitMQ
│   ├── definitions.json
│   └── rabbitmq.conf
└── images/                        # Placeholder para imagens
    ├── faces/
    └── teams/
```

## 🎓 Características Técnicas

### Conformidade com Requisitos

- ✅ 4 containers (RabbitMQ + Gerador + 2 Consumidores)
- ✅ Geração de carga rápida (6 msgs/seg)
- ✅ RabbitMQ com Topic Exchange
- ✅ Routing keys adequadas (face/team)
- ✅ Interface de administração habilitada
- ✅ Processamento lento para visualizar acúmulo
- ✅ IA embutida com biblioteca Smile
- ✅ Network compartilhada entre containers

### Implementações de IA

#### Análise de Sentimento

- Extração de características matemáticas da imagem
- Análise estatística com Smile (média, desvio padrão)
- Classificação baseada em nome + características
- Probabilidade de felicidade com ruído gaussiano

#### Identificação de Times

- Simulação de extração de características visuais (cores)
- Classificação por similaridade usando distância euclidiana
- Base de conhecimento de 10 times brasileiros
- Cálculo de confiança baseado em score de similaridade


---

_Sistema desenvolvido para demonstrar conceitos de sistemas distribuídos, containerização e integração de IA em arquiteturas de microsserviços._
