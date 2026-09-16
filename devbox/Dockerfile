FROM node:22-bookworm-slim

ARG DEVBOX_USER=node

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
    procps \
    sudo \
    ripgrep \
    fd-find \
    zoxide \
    vim \
    python3 \
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

# User-owned global package directories so the runtime user can install/update CLI packages.
ENV NPM_CONFIG_PREFIX=/home/${DEVBOX_USER}/.npm-global
ENV PNPM_HOME=/home/${DEVBOX_USER}/.local/share/pnpm
ENV PATH=$PNPM_HOME/bin:$PNPM_HOME:$NPM_CONFIG_PREFIX/bin:$PATH
RUN mkdir -p "$PNPM_HOME" "$NPM_CONFIG_PREFIX" \
    && chown -R ${DEVBOX_USER}:${DEVBOX_USER} /home/${DEVBOX_USER}/.local /home/${DEVBOX_USER}/.npm-global

# Corepack shims are installed system-wide; the pnpm store/home remains user-owned.
RUN corepack enable && corepack prepare pnpm@12.4.2 --activate

# pi CLI (https://pi.dev), installed in the runtime user's npm prefix for runtime updates.
USER ${DEVBOX_USER}
RUN npm install -g --ignore-scripts @earendil-works/pi-coding-agent
USER root

# Claude Code CLI, native installer (npm package is deprecated since v2.1.15).
# Installed as the runtime user so `claude update` can self-update the user-owned binary.
USER ${DEVBOX_USER}
RUN curl -fsSL https://claude.ai/install.sh | bash
USER root
RUN ln -sf /home/${DEVBOX_USER}/.local/bin/claude /usr/local/bin/claude

# agy CLI, always latest (installer has no version pin)
RUN curl -fsSL https://antigravity.google/cli/install.sh | bash \
    && if [ -f /root/.local/bin/agy ]; then cp /root/.local/bin/agy /usr/local/bin/agy; fi \
    && printf '#!/bin/sh\nexec /usr/local/bin/agy --dangerously-skip-permissions "$@"\n' \
        > /usr/local/bin/agyd \
    && chmod +x /usr/local/bin/agyd

# uv (Python package/venv manager), always latest
RUN curl -fsSL https://astral.sh/uv/install.sh | UV_INSTALL_DIR=/usr/local/bin sh

# Grant the runtime user passwordless sudo.
RUN printf '%s ALL=(ALL) NOPASSWD:ALL\n' "$DEVBOX_USER" > /etc/sudoers.d/${DEVBOX_USER} \
    && chmod 0440 /etc/sudoers.d/${DEVBOX_USER}

# Pre-create state directories so bind mounts retain a runtime-user-owned parent.
RUN mkdir -p /home/${DEVBOX_USER}/.gemini /home/${DEVBOX_USER}/.pi/agent /home/${DEVBOX_USER}/.claude \
    && chown -R ${DEVBOX_USER}:${DEVBOX_USER} /home/${DEVBOX_USER}

WORKDIR /workspace
CMD ["sleep", "infinity"]
