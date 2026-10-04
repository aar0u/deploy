# deploy

Docker image definitions built and published via GitHub Actions to `ghcr.io`.

## Projects

- [caddy-l4/](caddy-l4/) — minimal Caddy build with the [caddy-l4](https://github.com/mholt/caddy-l4) plugin, published as `ghcr.io/<owner>/caddy-l4`.
- [devbox/](devbox/) — persistent dev-container image (Node, common CLIs, non-root `developer` user) used as an agent/CI workspace base, published as `ghcr.io/<owner>/devbox`.
- [browser-box/](browser-box/) — headless browser & desktop runtime base image (Xvfb + Fluxbox + x11vnc + noVNC + Playwright Chromium + Supervisor) for multi-tenant automation and scrapers, published as `ghcr.io/<owner>/browser-box`.

Each subdirectory has its own README with build/usage details. CI workflows live in [.github/workflows/](.github/workflows/), one per project, each triggered only by changes under its own directory:
- `caddy-l4.yml` builds `caddy-l4`.
- `devbox.yml` builds `devbox`.
- `browser-box.yml` builds `browser-box`.
