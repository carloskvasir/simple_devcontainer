FROM node:22-bookworm-slim AS node-base
FROM ruby:3.3.6-bookworm-slim AS ruby-base

FROM debian:bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive

# Dependências de sistema para gems nativas e ferramentas de build
RUN apt-get update && apt-get install -y --no-install-recommends \
  build-essential \
  ca-certificates \
  curl \
  git \
  libpq-dev \
  libyaml-dev \
  libssl-dev \
  libreadline-dev \
  zlib1g-dev \
  procps \
  sudo \
  unzip \
  zsh \
  && rm -rf /var/lib/apt/lists/*

# Transplanta Node e Ruby para /usr/local sem compilação.
# Ordem importa: Node primeiro, Ruby por cima.
# Ruby vence conflitos e o Node mantém todos os caminhos relativos intactos
# (resolve o "Cannot find module '../lib/cli.js'" do npm).
COPY --from=node-base /usr/local /usr/local
COPY --from=ruby-base /usr/local /usr/local

# Cria usuário vscode
RUN useradd -m -s /usr/bin/zsh vscode \
  && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER vscode
WORKDIR /home/vscode

# Instala mise apenas para gerenciamento de versões por projeto (.mise.toml)
RUN curl https://mise.jdx.dev/install.sh | sh

# Ativação do mise nos shells interativos
RUN echo 'eval "$($HOME/.local/bin/mise activate bash)"' >> ~/.bashrc \
  && echo 'eval "$($HOME/.local/bin/mise activate zsh)"' >> ~/.zshrc

# Instala ferramentas de IA e Segurança
# 1. Claude Code (Anthropic)
# 2. OpenCode
# 3. Bitwarden CLI (bw)
# 4. Mailcatcher & Sidekiq (Gems)
RUN npm install -g @anthropic-ai/claude-code opencode @bitwarden/cli \
  && gem install mailcatcher sidekiq

CMD ["sleep", "infinity"]
