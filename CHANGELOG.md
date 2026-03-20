# Changelog

Todas as decisões técnicas relevantes deste projeto são documentadas aqui.
Formato baseado em [Keep a Changelog](https://keepachangelog.com/pt-BR/1.0.0/).

---

## [0.1.0] — 2026-03-20

### Adicionado

- **Devcontainer standalone** com `debian:bookworm` como base — escolhido por estabilidade e compatibilidade com gems nativas que exigem `build-essential`, `libpq-dev`, `libyaml-dev`
- **Mise** como gerenciador de versões de Ruby e Node — substituindo rbenv/asdf por ser mais rápido e ter suporte nativo a múltiplos runtimes num único arquivo de configuração
- **Ruby 3.3.6** e **Node 22** instalados via Mise no `Dockerfile`
- **Docker Compose** com serviços de infraestrutura: PostgreSQL 16, Redis 7, Mailcatcher, Adminer
- **Volumes nomeados** (`postgres_data`, `redis_data`) para persistência de dados entre reinicializações do Codespace
- **Claude Code** (`@anthropic-ai/claude-code`), **OpenCode** e **Bitwarden CLI** (`@bitwarden/cli`) instalados globalmente via npm
- **`bin/setup_dev`** — script de inicialização que verifica pré-requisitos, valida sessão do Bitwarden, sobe containers e aguarda PostgreSQL e Redis ficarem prontos
- **GitHub Actions** (`.github/workflows/devcontainer.yml`) — valida build do `Dockerfile` em PRs; publica imagem em `ghcr.io` no push à `main` usando `GITHUB_TOKEN` sem secrets extras
- **README** com guia de início rápido, tabela de serviços, fluxo do Bitwarden e instruções de CI

### Decisões técnicas

- **Bitwarden CLI isolado no shell do usuário** — o `bw login` é intencionalente manual e interativo. A `BW_SESSION` fica apenas na memória do shell; nenhuma chave é passada como variável de ambiente para os containers nem gravada em arquivo. Descartado o uso de `BW_CLIENTID`/`BW_CLIENTSECRET` via Codespaces Secrets por risco de exposição entre containers.

- **Serviço `app` removido do `docker-compose.yml`** — a versão inicial incluía um serviço `app` com `network_mode: service:db` para compartilhar o namespace de rede com o PostgreSQL. Isso falhou no podman-compose (rootless) com erro `permission denied` no netns. A solução foi remover o serviço `app` do Compose e deixar o devcontainer rodar de forma standalone via `build` no `devcontainer.json`. As portas de infra são expostas no host (`5432`, `6379`) e acessadas via `localhost`.

- **`DATABASE_URL` aponta para `localhost`** (não para o hostname `db`) — consequência da arquitetura standalone. Dentro do devcontainer, `localhost` alcança as portas expostas pelo Compose no host. Dentro do Adminer (que roda como container), o hostname correto é `db` (nome do serviço Compose).

- **Suporte a podman como fallback no `bin/setup_dev`** — o script detecta `docker` ou `podman` automaticamente. Em distribuições como Debian Trixie, `docker` é um alias para `podman` no zsh mas não é herdado pelo bash. O script usa `command -v` com fallback explícito para `podman`.

- **Cache do Buildx via GitHub Actions Cache** — `cache-from: type=gha` e `cache-to: type=gha,mode=max` reduzem o tempo de build em pushes subsequentes sem custo adicional de armazenamento externo.

- **Commits seguem Conventional Commits** (`feat:`, `chore:`, `fix:`, `ci:`, `docs:`) — padrão adotado a partir da v0.1.0.

---

## [0.1.4] — 2026-03-20

### Alterado

- **Imagens base trocadas por variantes `-slim`** — `node:22-bookworm-slim`, `ruby:3.3.6-bookworm-slim` e `debian:bookworm-slim`. Os stages de cópia não precisam de pacotes extras do sistema; a imagem final instala explicitamente tudo o que precisa via `apt-get`. Reduz as vulnerabilidades reportadas de 3 críticas + 70 altas para o mínimo necessário

---

## [0.1.3] — 2026-03-20

### Alterado

- **Multi-stage build com imagens oficiais** — `ruby:3.3.6-bookworm` como base e `node:22-bookworm` como stage de origem para o Node. Ruby e Node vêm pré-compilados; o tempo de build cai de ~10 minutos para segundos
- **COPY seletivo do Node** — em vez de `COPY --from=node-base /usr/local /usr/local` (que sobrescreveria o `/usr/local` inteiro do Ruby), copia apenas `bin/node`, `bin/npm`, `bin/npx` e `lib/node_modules`, evitando conflitos de arquivos entre as duas imagens
- **`mise use --global` removido** — mise não precisa mais compilar Ruby e Node; fica instalado apenas para gerenciamento de versões por projeto via `.mise.toml`. `MISE_DATA_DIR` e os `RUN mise use` foram removidos
- **`ENV PATH` dos shims removido** — desnecessário agora que Ruby e Node estão diretamente no `PATH` do sistema (`/usr/local/bin`)

---

## [0.1.2] — 2026-03-20

### Corrigido

- **`eval "$(mise activate bash)"` removido do `RUN`** — o Docker executa `RUN` com `/bin/sh` (dash no Debian), que não suporta sintaxe bash como `[[` e `export -a`. Substituído por `ENV PATH` apontando para o diretório de shims do mise (`~/.local/share/mise/shims`), que é suficiente para que `npm` e `gem` sejam encontrados nos passos de build subsequentes

---

## [0.1.1] — 2026-03-20

### Corrigido

- **`zsh` adicionado ao apt-get** — o `useradd -s /usr/bin/zsh` falhava silenciosamente porque zsh não estava instalado na imagem base
- **Caminho do binário do mise corrigido** — o script de instalação do mise coloca o binário em `~/.local/bin/mise`, não em `~/.local/share/mise/bin/mise`. O `MISE_DATA_DIR` controla apenas onde os runtimes (Ruby, Node) são armazenados. Todas as referências no `Dockerfile` foram corrigidas, incluindo as linhas de ativação no `.bashrc` e `.zshrc`

---

<!-- [0.1.1]: https://github.com/carloskvasir/simple_devcontainer/releases/tag/v0.1.1 -->
<!-- [0.1.0]: https://github.com/carloskvasir/simple_devcontainer/releases/tag/v0.1.0 -->
