FROM node:22-slim
WORKDIR /apps

ENV LANG=C.UTF-8
ENV LC_ALL=C.UTF-8
ENV TZ=Asia/Singapore
ENV DISPLAY=:99
ENV CHROMIUM_PATH=/usr/bin/chromium

# Install system dependencies: Chromium, ffmpeg, Xvfb, noVNC, fluxbox, supervisor, utilities
RUN apt-get update && apt-get install -y --no-install-recommends \
    chromium \
    ffmpeg \
    xvfb \
    x11vnc \
    novnc \
    websockify \
    fluxbox \
    tmux \
    vim \
    curl \
    git \
    ca-certificates \
    supervisor \
    procps \
    libatomic1 \
    && apt-get clean && rm -rf /var/lib/apt/lists/*

# Enable Corepack for pnpm management
RUN corepack enable pnpm

# Set tmux mouse mode enabled by default
RUN echo 'set -g mouse on' > /root/.tmux.conf

# Prepare directories for tenant apps and supervisor logs
RUN mkdir -p /apps /var/log/supervisor /etc/supervisor/conf.d

# Copy base supervisor configuration
COPY supervisord.conf /etc/supervisor/supervisord.conf

EXPOSE 8445

CMD ["/usr/bin/supervisord", "-n", "-c", "/etc/supervisor/supervisord.conf"]
