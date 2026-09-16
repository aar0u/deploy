FROM node:22-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV TZ=Asia/Singapore
# docker exec sessions have no login process to populate $SHELL from /etc/passwd,
# so tools that pick a shell by reading $SHELL (tmux, zellij) fall back to /bin/sh.
ENV SHELL=/bin/bash

# Install basic development tools and CLI utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    ca-certificates \
    curl \
    git \
    bash \
    procps \
    sudo \
    ripgrep \
    fd-find \
    zoxide \
    python3 \
    python3-pip \
    python3-venv \
    openssh-client \
    jq \
    tzdata \
    && ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && ln -sf $(which fdfind) /usr/local/bin/fd \
    && rm -rf /var/lib/apt/lists/*

# zellij, always latest (no apt package; install prebuilt musl binary from GitHub releases)
RUN curl -fsSL "$(curl -fsSL https://api.github.com/repos/zellij-org/zellij/releases/latest | jq -r '.assets[] | select(.name == "zellij-x86_64-unknown-linux-musl.tar.gz") | .browser_download_url')" \
    | tar -xz -C /usr/local/bin zellij

# fzf, always latest (Debian's apt package predates the --bash/--zsh integration flags _.zshrc relies on)
RUN arch=$(dpkg --print-architecture) \
    && curl -fsSL "$(curl -fsSL https://api.github.com/repos/junegunn/fzf/releases/latest | jq -r --arg arch "linux_$arch" '.assets[] | select(.name | test("^fzf-.*-" + $arch + ".tar.gz$")) | .browser_download_url')" \
    | tar -xz -C /usr/local/bin fzf

# starship prompt (avoids the git-prompt.sh fallback path in _.zshrc, which Debian's git package doesn't ship)
RUN curl -fsSL https://starship.rs/install.sh | sh -s -- --yes

# sops, always latest (no apt package)
RUN arch=$(dpkg --print-architecture) \
    && curl -fsSL -o /usr/local/bin/sops "$(curl -fsSL https://api.github.com/repos/getsops/sops/releases/latest | jq -r --arg arch "linux.$arch" '.assets[] | select(.name | test("^sops-v.*\\." + $arch + "$")) | .browser_download_url')" \
    && chmod +x /usr/local/bin/sops

# pnpm global bin dir ($PNPM_HOME/bin) must be on PATH before `pnpm add -g`
ENV PNPM_HOME=/pnpm
ENV PATH=$PNPM_HOME/bin:$PNPM_HOME:$PATH
RUN corepack enable && corepack prepare pnpm@12.4.2 --activate

# pi CLI (https://pi.dev), always latest
RUN pnpm add -g --ignore-scripts @earendil-works/pi-coding-agent

# agy CLI, always latest (installer has no version pin)
RUN curl -fsSL https://antigravity.google/cli/install.sh | bash \
    && if [ -f /root/.local/bin/agy ]; then cp /root/.local/bin/agy /usr/local/bin/agy; fi

# Base image's "node" user (UID 1000) gets passwordless sudo instead of a new user
RUN echo "node ALL=(ALL) NOPASSWD:ALL" >> /etc/sudoers.d/node \
    && chmod 0440 /etc/sudoers.d/node

# Pre-create state directories so bind mounts retain a node-owned parent.
RUN mkdir -p /home/node/.gemini /home/node/.pi/agent /home/node/.claude \
    && chown -R node:node /home/node

WORKDIR /workspace
CMD ["sleep", "infinity"]
