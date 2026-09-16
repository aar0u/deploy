# devbox

A persistent dev-container image used as a workspace base for agents/CI: Node 22 (package manager: pnpm, via corepack), common CLI tools (git, ripgrep, fd, python3, jq, ssh), the base image's non-root `node` user (UID 1000) granted passwordless sudo, and two AI coding agent CLIs preinstalled: [pi](https://pi.dev/) `pi` and [antigravity](https://antigravity.google/) `agy`.

The container has no entrypoint logic of its own — it just stays alive (`sleep infinity`) so something else (an agent runtime, a CI job, a devcontainer) can exec into it.

Timezone is fixed to `Asia/Singapore`.

## Usage

Both `pi` (via `pnpm add -g`, no version pin) and `agy` (via its official
installer) always install whatever they currently resolve as "latest" —
there's no build arg to pin either one. After building, CI reads the
actual installed versions back out of the image (`pi --version` /
`agy --version`) and uses them to tag it: `devbox:pi<version>-agy<version>`,
alongside a rolling `devbox:latest`.

```sh
docker build -t devbox -f devbox/Dockerfile devbox
docker run --rm -it devbox bash
```

Published for `linux/amd64` and `linux/arm64`.
