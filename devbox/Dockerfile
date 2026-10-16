FROM node:22-bookworm-slim

ARG DEVBOX_USER=node

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV TZ=Asia/Singapore
# docker exec sessions have no login process to populate $SHELL from /etc/passwd,
# so tools that pick a shell by reading $SHELL (such as tmux) fall back to /bin/sh.
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
    less \
    tmux \
    python3 \
    openssh-client \
    jq \
    tzdata \
    # Audio & media tools for recording/playback and Wayland clipboard via dx socket passthrough
    pulseaudio-utils \
    sox \
    libsox-fmt-pulse \
    wl-clipboard \
    && ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && ln -sf $(which fdfind) /usr/local/bin/fd \
    && rm -rf /var/lib/apt/lists/*

# fzf, always latest (Debian's apt package predates the --bash/--zsh integration flags _.zshrc relies on)
# The tag comes from the /releases/latest redirect, not the API, which rate-limits shared CI runners.
RUN arch=$(dpkg --print-architecture) \
    && tag=$(curl -fsSIL -o /dev/null -w '%{url_effective}' https://github.com/junegunn/fzf/releases/latest) \
    && tag=${tag##*/} \
    && curl -fsSL -o /tmp/fzf.tar.gz "https://github.com/junegunn/fzf/releases/download/$tag/fzf-${tag#v}-linux_$arch.tar.gz" \
    && tar -xzf /tmp/fzf.tar.gz -C /usr/local/bin fzf \
    && rm /tmp/fzf.tar.gz

# starship prompt (avoids the git-prompt.sh fallback path in _.zshrc, which Debian's git package doesn't ship)
RUN curl -fsSL https://starship.rs/install.sh | sh -s -- --yes

# sops, always latest (no apt package)
RUN arch=$(dpkg --print-architecture) \
    && tag=$(curl -fsSIL -o /dev/null -w '%{url_effective}' https://github.com/getsops/sops/releases/latest) \
    && tag=${tag##*/} \
    && curl -fsSL -o /usr/local/bin/sops "https://github.com/getsops/sops/releases/download/$tag/sops-$tag.linux.$arch" \
    && chmod +x /usr/local/bin/sops

# User-owned global package directories so the runtime user can install/update CLI packages.
ENV NPM_CONFIG_PREFIX=/home/${DEVBOX_USER}/.npm-global
ENV PNPM_HOME=/home/${DEVBOX_USER}/.local/share/pnpm
ENV PATH=$PNPM_HOME/bin:$PNPM_HOME:$NPM_CONFIG_PREFIX/bin:$PATH
RUN mkdir -p "$PNPM_HOME" "$NPM_CONFIG_PREFIX" \
    && chown -R ${DEVBOX_USER}:${DEVBOX_USER} /home/${DEVBOX_USER}/.local /home/${DEVBOX_USER}/.npm-global

# pnpm standalone installer (Node 25+ no longer ships corepack), into the user-owned PNPM_HOME.
USER ${DEVBOX_USER}
RUN curl -fsSL https://get.pnpm.io/install.sh | env PNPM_VERSION=12.4.2 SHELL=/bin/bash sh -
USER root

# Claude Code CLI, native installer (npm package is deprecated since v2.1.15).
# Installed as the runtime user so `claude update` can self-update the user-owned binary.
USER ${DEVBOX_USER}
RUN curl -fsSL https://claude.ai/install.sh | bash
USER root
RUN ln -sf /home/${DEVBOX_USER}/.local/bin/claude /usr/local/bin/claude

# OpenCode CLI, native installer into the runtime user's ~/.opencode/bin so `opencode upgrade` can self-update.
USER ${DEVBOX_USER}
RUN curl -fsSL https://opencode.ai/v2/install | bash
USER root
RUN ln -sf /home/${DEVBOX_USER}/.opencode/bin/opencode /usr/local/bin/opencode

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
RUN mkdir -p /home/${DEVBOX_USER}/.gemini /home/${DEVBOX_USER}/.claude \
        /run/user/$(id -u ${DEVBOX_USER}) \
    && chown -R ${DEVBOX_USER}:${DEVBOX_USER} /home/${DEVBOX_USER} /run/user/$(id -u ${DEVBOX_USER})

WORKDIR /home/${DEVBOX_USER}
CMD ["sleep", "infinity"]
