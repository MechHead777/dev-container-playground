# dev-container-playground

A minimal DevPod/dev container template: Ubuntu 24.04 base image with [mise](https://mise.jdx.dev) pre-installed, so a new dev container trusts and installs the project's `mise.toml` runtimes automatically on creation instead of me doing it by hand every time.

## What's in it

- `.devcontainer/Dockerfile` — pulls the `mise` binary straight from its own container image and wires it into both `bash` and `zsh`
- `.devcontainer/devcontainer.json` — builds from that Dockerfile and runs `scripts/setup` once the container's created
- `scripts/setup` — trusts and installs the project's `mise.toml`, so language runtimes are pinned per-project instead of living on the host

## Using this as a template

`scripts/setup` currently hardcodes `dev-container-playground` as the workspace ID. Swap that for whatever the actual project directory is named before reusing this in another repo.
