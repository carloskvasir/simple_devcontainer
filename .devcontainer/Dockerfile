FROM node:22-bookworm AS node-base
FROM ruby:3.3.6-bookworm

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

# Copia apenas os binários do Node (sem sobrescrever /usr/local inteiro do Ruby)
COPY --from=node-base /usr/local/bin/node /usr/local/bin/node
COPY --from=node-base /usr/local/bin/npm  /usr/local/bin/npm
COPY --from=node-base /usr/local/bin/npx  /usr/local/bin/npx
COPY --from=node-base /usr/local/lib/node_modules /usr/local/lib/node_modules

# Cria usuário vscode
RUN useradd -m -s /usr/bin/zsh vscode \
  && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER vscode
WORKDIR /home/vscode

# Instala mise apenas para gerenciamento de versões por projeto (.mise.toml)
# Ruby e Node já estão disponíveis via imagens oficiais — mise não compila nada
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
