FROM node:22-bookworm-slim

ENV DEBIAN_FRONTEND=noninteractive
ENV LANG=C.UTF-8
ENV TZ=Asia/Singapore

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
    python3 \
    python3-pip \
    python3-venv \
    openssh-client \
    jq \
    tzdata \
    && ln -sf /usr/share/zoneinfo/$TZ /etc/localtime && echo $TZ > /etc/timezone \
    && ln -sf $(which fdfind) /usr/local/bin/fd \
    && rm -rf /var/lib/apt/lists/*

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

WORKDIR /workspace
CMD ["sleep", "infinity"]
