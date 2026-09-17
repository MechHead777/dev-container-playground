# Dev Container Playground

A small playground for developing with DevPod, Docker, and dev containers.

The development environment is defined under `.devcontainer/`, and project tooling is managed with mise. When a workspace starts, DevPod clones my [dotfiles](https://github.com/MechHead777/dotfiles) repo and runs its `setup` script, which applies my shell, editor, and CLI tools with chezmoi.

## Part of the stateless workstation

This repo is one of three that set up my environment from scratch:

- [dotfiles](https://github.com/MechHead777/dotfiles): shell, editor, and CLI tools (chezmoi + mise), used on Arch, macOS, WSL, and inside containers
- dev-container-playground (this repo): project environments with DevPod
- arch-bootstrap (planned): Arch install, pacman packages, system services, then dotfiles and DevPod setup

## Repository Structure

- `.devcontainer/devcontainer.json`: points DevPod at the Dockerfile and runs `scripts/setup` after the container is created
- `.devcontainer/Dockerfile`: builds from the Ubuntu 24.04 Dev Container base image and installs mise
- `scripts/setup`: trusts this repo's `mise.toml` and installs the tools listed in it
- `mise.toml`: project tools go here, pinned to the versions this project needs. It is blank for now.

# New Machine Setup

## 1. Install Prerequisites

**Warning: members of the `docker` group can run containers as root, which is effectively root access to the host. Only add users you would trust with `sudo`.**

First, install OpenSSH, Git, and Docker with your package manager.

Then, enable and start Docker: `sudo systemctl enable --now docker`

Then, add your user to the `docker` group: `sudo usermod -aG docker "$USER"`

After that, log out and back in so the group change takes effect.

Finally, verify Docker is working: `docker run hello-world`

> If this fails with a permission error, it's often because the session was not restarted after adding the group.

## 2. Set Up SSH

SSH is only needed for pushing to GitHub from inside the container. Cloning this repo and the dotfiles repo works over HTTPS.

First, check for an existing key: `ls ~/.ssh/id_ed25519`

If there isn't one, create it: `ssh-keygen -t ed25519`

Then, add `~/.ssh/id_ed25519.pub` to your GitHub account.

After that, start the SSH agent and load the key:

```sh
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Finally, verify GitHub accepts the key: `ssh -T git@github.com`

## 3. Clone This Repo

```sh
git clone https://github.com/MechHead777/dev-container-playground.git
cd dev-container-playground
```

## 4. Install DevPod

Install DevPod using the [DevPod installation instructions](https://devpod.sh/docs/getting-started/install).

Then, verify the install: `devpod version`

## 5. Configure DevPod

First, add and select the Docker provider:

```sh
devpod provider add docker
devpod provider use docker
```

Then, set `none` as the default IDE so workspaces open in the terminal: `devpod ide use none`

Then, point DevPod at the dotfiles repo and its setup script:

```sh
devpod context set-options \
  -o DOTFILES_URL=https://github.com/MechHead777/dotfiles \
  -o DOTFILES_SCRIPT=setup
```

> Setting `DOTFILES_SCRIPT` explicitly means a failing setup script shows up as an error. Without it, DevPod silently falls back to symlinking the repo's dotfiles into `~`.

Finally, enable SSH agent forwarding so you can push from inside the container:

```sh
devpod context set-options \
  -o SSH_AGENT_FORWARDING=true \
  -o SSH_ADD_PRIVATE_KEYS=true
```

> If DevPod reports `ssh-agent is not started`, start the agent and load your key as shown in step 2, then run `devpod up .` again.

## 6. Start the Workspace

From the repo, start the workspace: `devpod up .`

> If the dotfiles fail to install, run `devpod up . --debug` and look for the lines about cloning the dotfiles repo and running `./setup`.

After it's finished building, connect to it: `devpod ssh .`

## 7. Verify the Workspace

Once inside, check that the dotfiles were applied:

```sh
chezmoi status            # should print nothing
which bat eza starship    # should point into ~/.local/share/mise
```

> DevPod clones the dotfiles into `~/dotfiles` only once and never pulls. If a workspace is showing old dotfiles, it's often because it was created before the latest push. Delete it with `devpod delete <workspace>` and start a new one. You can also rerun from the desired directory with 'devpod up . --recreate' to save yourself an extra step.
