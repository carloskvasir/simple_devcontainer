FROM debian:bookworm

ENV DEBIAN_FRONTEND=noninteractive
ENV MISE_DATA_DIR=/home/vscode/.local/share/mise

# Dependências de sistema para Rails, Bitwarden e ferramentas de build
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

# Cria usuário vscode
RUN useradd -m -s /usr/bin/zsh vscode \
  && echo "vscode ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers

USER vscode
WORKDIR /home/vscode

# Instala Mise
RUN curl https://mise.jdx.dev/install.sh | sh

# Instala Ruby 3.3.6 e Node 22 via Mise
RUN /home/vscode/.local/bin/mise use --global ruby@3.3.6 \
  && /home/vscode/.local/bin/mise use --global node@22

# Instala ferramentas de IA e Segurança
# 1. Claude Code (Anthropic)
# 2. OpenCode
# 3. Bitwarden CLI (bw)
# 4. Mailcatcher & Sidekiq (Gems)
RUN eval "$(/home/vscode/.local/bin/mise activate bash)" \
  && npm install -g @anthropic-ai/claude-code opencode @bitwarden/cli \
  && gem install mailcatcher sidekiq

# Ativação do Mise nos shells
RUN echo 'eval "$($HOME/.local/bin/mise activate bash)"' >> ~/.bashrc \
  && echo 'eval "$($HOME/.local/bin/mise activate zsh)"' >> ~/.zshrc

CMD ["sleep", "infinity"]
