# Dev Container Playground

 A small playground for developing with DevPod, Docker, and devcontainers.

 The development environment is defined under `.devcontainer/`, and development tooling is managed with mise.

 ## Repository Structure

- `.devcontainer/` — Defines the development container and configuration
- `Dockerfile` — Builds from the Ubuntu 24.04 Dev Container base image and installs mise
- `scripts/setup` — Trusts the repo's `mise.toml` and installs configured tools
- `mise.toml` — Intentionally blank for now; tools can be added later

 DevPod uses my private dotfiles repository to configure the container environment.

 # New Machine Setup

 ## 1\. Install Prerequisites

 Install:

- OpenSSH
- Git
- Docker

 Enable and start Docker:

```
sudo systemctl enable --now docker
```

 Add your user to the Docker group:

```
sudo usermod -aG docker "$USER"
```

 Log out and back in after this.

 Verify Docker:

```
docker run hello-world
```

 ## 2\. Set Up SSH

 Make sure you have an SSH key:

```
ls ~/.ssh/id_ed25519
```

 If needed, create one:

```
ssh-keygen -t ed25519
```

 Add `~/.ssh/id_ed25519.pub` to GitHub.

 Start the SSH agent and load the key:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

 Verify:

```
ssh-add -l
ssh -T git@github.com
```

 ## 3\. Clone This Repo

 Use SSH:

```
git clone git@github.com:MechHead777/dev-container-playground.git
cd dev-container-playground
```

 ## 4\. Install DevPod

 Install DevPod using the installation instructions from the [DevPod documentation](<https://devpod.sh/docs/getting-started/install>).

 Verify:

```
devpod version
```

 ## 5\. Configure DevPod

 Add and select the Docker provider:

```
devpod provider add docker
devpod provider use docker
```

 Set `none` as the default IDE:

```
devpod ide use none
```

 Set the dotfiles repository:

```
devpod context set-options \
  -o DOTFILES_URL=git@github.com:MechHead777/dotfiles.git
```

 Enable SSH agent forwarding:

```
devpod context set-options \
  -o SSH_AGENT_FORWARDING=true \
  -o SSH_ADD_PRIVATE_KEYS=true
```

 ## 6\. Start the Workspace

 From the repo:

```
devpod up .
```

 Since `none` is configured as the default IDE, no `--ide none` argument is needed.

 ## 7\. Verify SSH + Dotfiles

 Before starting DevPod, verify that all of these work:

```
ssh-add -l
ssh -T git@github.com
git ls-remote git@github.com:MechHead777/dotfiles.git
```

 When the workspace starts, DevPod should be able to clone and install:

```
git@github.com:MechHead777/dotfiles.git
```

 ## Troubleshooting

 ### Dotfiles Fail to Install

 Verify SSH access:

```
ssh-add -l
ssh -T git@github.com
git ls-remote git@github.com:MechHead777/dotfiles.git
```

 Then retry with debug output:

```
devpod up . --debug
```

 ### `ssh-agent` Is Not Started

 If DevPod reports:

```
ssh-agent is not started
```

 Start the SSH agent and add your key:

```
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

 Then retry:

```
devpod up .
```

