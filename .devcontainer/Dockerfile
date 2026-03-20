# ── Build stages: runtimes pré-compilados (zero compile) ─────────────────────
FROM ruby:3.3-slim AS ruby-base
FROM node:22-slim  AS node-base

# ── Imagem final ──────────────────────────────────────────────────────────────
FROM debian:trixie-slim

ENV DEBIAN_FRONTEND=noninteractive

# Dependências de sistema (gems nativas, build de extensões C, ferramentas)
RUN apt-get update && apt-get install -y --no-install-recommends \
  build-essential \
  ca-certificates \
  curl \
  git \
  libyaml-dev \
  libpq-dev \
  procps \
  sudo \
  unzip \
  zsh \
  && rm -rf /var/lib/apt/lists/*

# Transplante de runtimes para /usr/local sem compilação.
# Ordem: Node primeiro → Ruby por cima.
# Ruby vence conflitos; Node mantém caminhos relativos intactos (fix npm cli.js).
COPY --from=node-base /usr/local /usr/local
COPY --from=ruby-base /usr/local /usr/local

# Cria usuário vscode com zsh e sudo sem senha
RUN useradd -m -s /usr/bin/zsh vscode \
  && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER vscode
WORKDIR /home/vscode

# Expõe o diretório de binários do mise ao PATH para todos os RUN seguintes
ENV PATH="/home/vscode/.local/bin:$PATH"

# Instala mise
RUN curl https://mise.jdx.dev/install.sh | sh

# Instala opencode via backend ubi do mise (evita instabilidades de npm 404)
RUN mise use -g ubi:sst/opencode

# Ativação do mise nos shells interativos
RUN echo 'eval "$($HOME/.local/bin/mise activate bash)"' >> ~/.bashrc \
  && echo 'eval "$($HOME/.local/bin/mise activate zsh)"' >> ~/.zshrc

# Ferramentas npm e gems
RUN npm install -g @anthropic-ai/claude-code @bitwarden/cli \
  && gem install mailcatcher sidekiq

CMD ["sleep", "infinity"]
