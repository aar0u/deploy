# deploy

Docker image definitions built and published via GitHub Actions to `ghcr.io`.

## Projects

- [caddy-l4/](caddy-l4/) — minimal Caddy build with the [caddy-l4](https://github.com/mholt/caddy-l4) plugin, published as `ghcr.io/<owner>/caddy-l4`.
- [devbox/](devbox/) — persistent dev-container image (Node, common CLIs, non-root `developer` user) used as an agent/CI workspace base, published as `ghcr.io/<owner>/devbox`.

Each subdirectory has its own README with build/usage details. CI workflows live in [.github/workflows/](.github/workflows/), one per project, each triggered only by changes under its own directory:
- `caddy-l4.yml` builds `caddy-l4`.
- `devbox.yml` builds `devbox`.
