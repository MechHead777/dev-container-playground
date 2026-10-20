# dev-container-playground

Per-project development environments with DevPod and Docker. The container image defines the base environment, mise manages project tools, and the [dotfiles](https://github.com/MechHead777/dotfiles) repo supplies the shell, editor, and CLI configuration.

## Part of the stateless workstation

The workstation setup is split into three repos, used in this order:

- [arch-bootstrap](https://github.com/MechHead777/arch-bootstrap): OS layer: Arch installation, system packages, and services.
- [dotfiles](https://github.com/MechHead777/dotfiles): User layer: shell, editor, and CLI tools with chezmoi and mise.
- dev-container-playground (this repo): Project layer: per-project environments with DevPod and Docker.

## What's Included

| Area | Configuration |
|---|---|
| Container | Ubuntu 24.04 Dev Container base image with mise installed |
| User environment | Dotfiles applied through chezmoi when DevPod creates the workspace |
| Project tools | Versions declared in the project's `mise.toml` and installed by `scripts/setup` |

## Setup

### Arch with arch-bootstrap

Follow [arch-bootstrap](https://github.com/MechHead777/arch-bootstrap#setup) to install the prerequisites and DevPod, apply the host dotfiles, and configure the Docker provider and dotfiles options.

> If the first bootstrap run deferred DevPod configuration, log out and back in, then run `./bootstrap.sh --only devpod` from the bootstrap repo.

After bootstrap is complete, continue with [Usage](#usage). For GitHub pushes from inside the container, also complete [SSH access](#ssh-access-optional).

### Manual setup on another Linux host

#### 1. Install prerequisites

**Warning: Docker group membership grants effective root access to the host. Only add users you would trust with `sudo`.**

First, install OpenSSH, Git, and Docker with your package manager. Then, enable Docker and add your user to its group:

```sh
sudo systemctl enable --now docker
sudo usermod -aG docker "$USER"
```

After logging out and back in, verify Docker access:

```sh
docker run hello-world
```

> A permission error often means the session wasn't restarted after adding the group.

#### 2. Install DevPod

Follow the [DevPod installation instructions](https://devpod.sh/docs/getting-started/install), then verify the installation:

```sh
devpod version
```

#### 3. Configure DevPod

First, add and select the Docker provider:

```sh
devpod provider add docker
devpod provider use docker
```

Then, select the terminal-only workflow:

```sh
devpod ide use none
```

Point DevPod at the dotfiles repo and its setup script:

```sh
devpod context set-options \
  -o DOTFILES_URL=https://github.com/MechHead777/dotfiles \
  -o DOTFILES_SCRIPT=setup
```

> Setting `DOTFILES_SCRIPT` explicitly makes a failing setup script visible. Without it, DevPod falls back to symlinking the repo's dotfiles into the home directory.

Finally, configure SSH agent forwarding for GitHub pushes from inside the container:

```sh
devpod context set-options \
  -o SSH_AGENT_FORWARDING=true \
  -o SSH_ADD_PRIVATE_KEYS=true
```

### SSH access (optional)

Cloning this repo and the dotfiles repo works over HTTPS. Set up SSH if you need to push to GitHub from inside the container.

First, check for an existing key:

```sh
ls ~/.ssh/id_ed25519
```

If there isn't one, create it:

```sh
ssh-keygen -t ed25519
```

Then, add `~/.ssh/id_ed25519.pub` to your GitHub account. Start the SSH agent and load the key in the host shell used to launch DevPod:

```sh
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Finally, verify GitHub access:

```sh
ssh -T git@github.com
```

> GitHub returns status `1` after a successful SSH authentication. If DevPod reports `ssh-agent is not started`, start the agent and load the key in the current host shell, then run `devpod up .` again.

## Usage

### Clone the project

```sh
git clone https://github.com/MechHead777/dev-container-playground.git
cd dev-container-playground
```

### Start and connect

Start the workspace from the repo directory:

```sh
devpod up .
```

> If dotfiles setup fails, run `devpod up . --debug` and look for the clone and `./setup` output.

After the build finishes, connect to the container:

```sh
devpod ssh .
```

### Refresh workspace dotfiles

DevPod clones the dotfiles into `~/dotfiles` once and doesn't pull updates automatically. To pick up newer dotfiles, run this from the project directory on the host:

```sh
devpod up . --recreate
```

Alternatively, delete the workspace with `devpod delete <workspace>` and start a new one.

## Verification

Inside the container, check the user, tool paths, and managed files:

```sh
id -un
command -v bat eza starship mise chezmoi
chezmoi status
```

The user should be `vscode`, all five commands should resolve, and `chezmoi status` should print nothing when there are no local changes. The bat, eza, and starship paths should point into `~/.local/share/mise`.

## Repository Layout

| Path | Purpose |
|---|---|
| `.devcontainer/devcontainer.json` | Selects the Dockerfile and runs `scripts/setup` after container creation |
| `.devcontainer/Dockerfile` | Builds from the Ubuntu 24.04 Dev Container base image and installs mise |
| `scripts/setup` | Trusts the project's `mise.toml` and installs its tools |
| `mise.toml` | Project-specific tool versions; currently blank until tools are added |

## Design Decisions

| Decision | Choice | Reasoning |
|---|---|---|
| Workspace runtime | DevPod with Docker | Uses the Docker provider configured by the workstation bootstrap |
| User configuration | Shared dotfiles repo | Applies the same shell, editor, and CLI setup inside each workspace |
| Project tools | Per-project mise configuration | Keeps project versions in the repo's `mise.toml` |
| Editor connection | Default IDE set to `none` | Keeps the workflow in the terminal through `devpod ssh .` |
