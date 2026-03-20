# simple_devcontainer

Ambiente de desenvolvimento Rails 8 reproduzível, baseado em **GitHub Codespaces** e **Docker Compose**. Roda igual em qualquer máquina ou Codespace sem configuração manual.

## O que está incluído

| Serviço / Ferramenta | Versão           | Função                                       |
| -------------------- | ---------------- | -------------------------------------------- |
| Ruby                 | 3.3.6 (via Mise) | Runtime principal                            |
| Node                 | 22 (via Mise)    | Assets / JS toolchain                        |
| PostgreSQL           | 16               | Banco de dados (dados persistidos em volume) |
| Redis                | 7                | Cache e filas (Sidekiq)                      |
| Mailcatcher          | latest           | Captura e-mails em desenvolvimento           |
| Adminer              | latest           | Client web para o banco                      |
| Claude Code          | latest           | AI no terminal (Anthropic)                   |
| OpenCode             | latest           | Agentes de software                          |
| Bitwarden CLI        | latest           | Gerenciamento seguro de segredos             |

## Início rápido

### 1. Abrir no Codespace

Clique em **Code → Codespaces → Create codespace on main** no GitHub.
O devcontainer sobe automaticamente com todas as ferramentas instaladas.

### 2. Subir os serviços

```bash
bin/setup_dev
```

O script verifica pré-requisitos, sobe os containers e aguarda PostgreSQL e Redis ficarem prontos.

### 3. Criar e migrar o banco

```bash
rails db:create db:migrate
```

### 4. Iniciar o servidor

```bash
rails server
```

A porta 3000 é encaminhada automaticamente pelo Codespace. Acesse via **Ports → 3000** na interface do VS Code.

---

## Portas e serviços

| Serviço             | URL / Endereço        |
| ------------------- | --------------------- |
| Rails app           | http://localhost:3000 |
| PostgreSQL          | localhost:5432        |
| Redis               | localhost:6379        |
| Mailcatcher (UI)    | http://localhost:1080 |
| Adminer (DB client) | http://localhost:8080 |

---

## Bitwarden — gerenciamento de segredos

O Bitwarden CLI roda **isolado no seu shell**. Nenhuma chave é passada para os containers nem gravada em arquivo.

### Primeiro acesso

```bash
bw login
```

Abre o fluxo interativo de autenticação (email + senha + 2FA).

### Desbloquear em sessões futuras

```bash
export BW_SESSION=$(bw unlock --raw)
```

### Usar um segredo no shell

```bash
# Exemplo: pegar uma API key pelo nome do item
bw get password "minha-api-key"

# Usar diretamente como variável de ambiente (não persiste, só na sessão)
export STRIPE_SECRET_KEY=$(bw get password "stripe-prod")
```

> O `bin/setup_dev` verifica se você tem sessão ativa e instrui o desbloqueio se necessário, mas não bloqueia a inicialização dos containers.

---

## Variáveis de ambiente

Configuradas automaticamente pelo `devcontainer.json`:

| Variável       | Valor padrão                                    |
| -------------- | ----------------------------------------------- |
| `DATABASE_URL` | `postgres://postgres:password@db:5432/postgres` |
| `REDIS_URL`    | `redis://redis:6379/1`                          |

Para sobrescrever localmente, crie um arquivo `.env` na raiz (já incluído no `.gitignore` por padrão em projetos Rails).

---

## Ferramentas de AI

### Claude Code

```bash
claude
```

Interface de linha de comando para o Claude (Anthropic). Requer a variável `ANTHROPIC_API_KEY` — use o Bitwarden para injetá-la:

```bash
export ANTHROPIC_API_KEY=$(bw get password "anthropic-api-key")
claude
```

### OpenCode

```bash
opencode
```

---

## Comandos úteis

```bash
# Ver logs de um serviço
docker compose logs -f db
docker compose logs -f redis

# Parar todos os containers
docker compose down

# Derrubar e apagar volumes (reset total do banco)
docker compose down -v

# Acessar o banco via psql
docker compose exec db psql -U postgres

# Acessar o Redis
docker compose exec redis redis-cli
```

---

## GitHub Actions — CI da imagem

O workflow em [`.github/workflows/devcontainer.yml`](.github/workflows/devcontainer.yml) roda automaticamente quando `Dockerfile`, `devcontainer.json` ou `docker-compose.yml` são alterados:

| Evento         | O que acontece                                                                    |
| -------------- | --------------------------------------------------------------------------------- |
| Pull Request   | Faz o build sem publicar — valida que o `Dockerfile` não está quebrado            |
| Push na `main` | Faz o build e publica a imagem em `ghcr.io` com as tags `latest` e `sha-<commit>` |

A autenticação usa o `GITHUB_TOKEN` gerado automaticamente pelo Actions — nenhum secret extra precisa ser configurado.

### Usar a imagem pré-construída no Codespace

Após o primeiro push na `main`, você pode substituir o build local pela imagem publicada no `devcontainer.json`:

```json
// Troque o bloco "build" por "image":
"image": "ghcr.io/carloskvasir/simple_devcontainer/devcontainer:latest"
```

Isso reduz o tempo de abertura do Codespace significativamente — a imagem já vem pronta, sem precisar compilar Ruby e instalar gems na inicialização.

### Deletar a imagem

Acesse **github.com → seu perfil → Packages → devcontainer → Delete this package**.
A imagem é removida completamente, sem rastro público.

---

## Estrutura do repositório

```
.
├── .devcontainer/
│   ├── Dockerfile          # Imagem do ambiente de desenvolvimento
│   └── devcontainer.json   # Configuração do Codespace (extensões, portas, etc.)
├── .github/
│   └── workflows/
│       └── devcontainer.yml  # CI: valida no PR, publica imagem no push à main
├── bin/
│   └── setup_dev           # Script de inicialização do ambiente
└── docker-compose.yml      # PostgreSQL, Redis, Mailcatcher, Adminer
```

---

## Usando localmente (fora do Codespace)

Requer Docker Engine (ou Podman) instalado. O script detecta automaticamente.

```bash
git clone <repo>
cd simple_devcontainer
bin/setup_dev
```

> Mise, Ruby e Node precisam estar instalados localmente se você for desenvolver fora do devcontainer. Dentro do Codespace tudo já está disponível.
