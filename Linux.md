# Linux Setup for Software Engineering

Step-by-step guide to take a fresh **Ubuntu** machine from zero to a fully featured software engineering box.

Targets **Ubuntu 24.04 LTS** (works on 22.04 LTS too — notes call out the differences). Install commands assume `bash`; **zsh** becomes your daily shell after Phase 2. Anything with `sudo` needs an admin account.

---

## What you get


| Category          | Tools                                                   |
| ----------------- | ------------------------------------------------------- |
| Security & access | Bitwarden, VPN client (Clash Verge / sing-box), GitHub  |
| Build base        | build-essential, curl, git, common dev headers          |
| Package managers  | apt (primary), Homebrew for Linux (optional)            |
| Shell             | zsh, Oh My Zsh, Powerlevel10k, fzf, fd                  |
| Languages         | Node (nvm), Bun, Python (pyenv), Go, Rust               |
| Containers        | Docker Engine + Compose plugin (native, no VM)          |
| DevOps CLI        | git, GitHub CLI, kubectl, Terraform                     |
| GUI apps          | Cursor, Chrome, Postman, Obsidian, Ollama + Open WebUI, Terax |
| AI code intel     | CodeGraph (local MCP knowledge graph for agents)        |
| AI docs           | Context7 (up-to-date library docs for agents)           |


---



## Before you start

- Connect to the network.
- Confirm your user is in `sudo`: `groups | grep sudo`.
- Plug in power — some steps take a while.

Update the system first:

```bash
sudo apt update && sudo apt upgrade -y
```

Estimated time: **1–2 hours**, mostly downloads.

---



## Phase 0 — Essentials (before dev tools)

Do these first. You need passwords, network access, and a compiler toolchain before everything else.

### Step 0.1 — Bitwarden (password manager)

```bash
sudo snap install bitwarden
```

Prefer no snap? Download the `.deb` or AppImage from [bitwarden.com/download](https://bitwarden.com/download/), then:

```bash
sudo apt install -y ./Bitwarden-*-amd64.deb
```

1. Open Bitwarden → sign in.
2. Install the **Chrome** or **Firefox** browser extension when prompted.
3. You now have your PASSWORDS!



### Step 0.2 — VPN client

Shadowrocket is iOS/macOS only. On Ubuntu, pick one of these:

| Client                | Install                                                                                    |
| --------------------- | ------------------------------------------------------------------------------------------ |
| **Clash Verge Rev**   | `.deb` from [GitHub releases](https://github.com/clash-verge-rev/clash-verge-rev/releases)  |
| **sing-box** (CLI)    | [sing-box docs](https://sing-box.sagernet.org/installation/package-manager/)                |
| **v2rayA** (web UI)   | `.deb` from [v2raya.org](https://v2raya.org/en/docs/prologue/installation/)                 |

Example (Clash Verge Rev):

```bash
sudo apt install -y ./clash-verge_*_amd64.deb
```

1. Import your subscription URL.
2. Enable system proxy or TUN mode.
3. Verify: `curl -s https://ipinfo.io/ip`

> Use VPN before signing into sensitive accounts.



### Step 0.3 — Sign in to GitHub

Go to github.com → use Bitwarden → sign in.

You will configure SSH keys and the `gh` CLI later in Phase 3.

### Step 0.4 — Build toolchain

Ubuntu's equivalent of Xcode Command Line Tools. Required for native compilation and for building Python versions with pyenv.

```bash
sudo apt install -y build-essential curl wget git ca-certificates gnupg \
  pkg-config libssl-dev unzip zip software-properties-common
```

**Verify:**

```bash
gcc --version    # gcc (Ubuntu 13.x)
make --version
git --version    # git version 2.x
```

---



## Phase 1 — Package managers

`apt` is already installed and is the primary package manager here. Homebrew for Linux is optional — useful when a tool ships a fresher formula than Ubuntu's repos.

### Step 1.1 — (Optional) Install Homebrew for Linux

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Homebrew installs to `/home/linuxbrew/.linuxbrew`.

### Step 1.2 — Add Homebrew to your shell

The installer prints these commands — run them:

```bash
echo >> ~/.bashrc
echo 'eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"' >> ~/.bashrc
eval "$(/home/linuxbrew/.linuxbrew/bin/brew shellenv)"
```

After Phase 2, add the same `eval` line to `~/.zshrc`.

**Verify:**

```bash
brew --version
brew doctor
```

> The rest of this guide uses `apt` and official installers, so Homebrew stays optional.

---



## Phase 2 — Shell setup

A good shell saves hours every week. Unlike macOS, Ubuntu ships **bash** — install zsh first.

### Step 2.1 — Install zsh and make it default

```bash
sudo apt install -y zsh
chsh -s "$(which zsh)"
```

Log out and back in (or reboot) for the shell change to take effect.

### Step 2.2 — Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

### Step 2.3 — Install zsh plugins

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```



### Step 2.4 — Install Powerlevel10k theme

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```

Powerlevel10k needs a Nerd Font. Install MesloLGS NF:

```bash
mkdir -p ~/.local/share/fonts
cd ~/.local/share/fonts
for style in "Regular" "Bold" "Italic" "Bold%20Italic"; do
  wget -q "https://github.com/romkatv/powerlevel10k-media/raw/master/MesloLGS%20NF%20${style}.ttf"
done
fc-cache -f
cd -
```

Then set **MesloLGS NF** as the font in your terminal profile's settings.

### Step 2.5 — Install fzf and fd

```bash
sudo apt install -y fzf fd-find
```

Ubuntu ships fd as **`fdfind`** (the name `fd` is taken by another package). Link it:

```bash
mkdir -p ~/.local/bin
ln -sf "$(which fdfind)" ~/.local/bin/fd
```

> Want the newest fzf with all key bindings? Install from source instead:
> `git clone --depth 1 https://github.com/junegunn/fzf.git ~/.fzf && ~/.fzf/install`

### Step 2.6 — Configure ~/.zshrc

Edit `~/.zshrc`. At minimum, set:

```bash
# Theme
ZSH_THEME="powerlevel10k/powerlevel10k"

# Plugins
plugins=(
  git
  docker
  docker-compose
  golang
  npm
  fzf
  z
  extract
  zsh-autosuggestions
  zsh-syntax-highlighting
)
```

Add language paths (fill in as you install each runtime in Phase 5):

```bash
# Local binaries (fd symlink, pipx, codegraph)
export PATH="$HOME/.local/bin:$PATH"

# Bun
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"

# Go
export GOPATH="$HOME/go"
export PATH="$PATH:/usr/local/go/bin:$GOPATH/bin"

# Rust
[ -f "$HOME/.cargo/env" ] && . "$HOME/.cargo/env"

# pyenv (after Python install)
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - zsh)"

# nvm (after Node install — see Step 5.1)
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"

# fzf
source /usr/share/doc/fzf/examples/key-bindings.zsh 2>/dev/null
source /usr/share/doc/fzf/examples/completion.zsh 2>/dev/null
export FZF_DEFAULT_OPTS="--height 40% --layout=reverse --border"
export FZF_DEFAULT_COMMAND="fd --type f --hidden --exclude .git"
export FZF_CTRL_T_COMMAND="$FZF_DEFAULT_COMMAND"

# Editor
export EDITOR="cursor --wait"   # or: nvim / code
export VISUAL="$EDITOR"
```

Reload:

```bash
source ~/.zshrc
```

Powerlevel10k runs a setup wizard on first launch — follow the prompts or run `p10k configure` later.

---



## Phase 3 — Git and GitHub CLI



### Step 3.1 — Configure Git identity

```bash
git config --global user.name "Your Name"
git config --global user.email "you@example.com"
git config --global init.defaultBranch main
git config --global pull.rebase false
git config --global core.editor "cursor --wait"
```

Use the same email as your GitHub account.

### Step 3.2 — Generate SSH key for GitHub

```bash
ssh-keygen -t ed25519 -C "you@example.com" -f ~/.ssh/id_ed25519
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
```

Add to `~/.ssh/config`:

```
Host github.com
  AddKeysToAgent yes
  IdentityFile ~/.ssh/id_ed25519
```

```bash
chmod 700 ~/.ssh && chmod 600 ~/.ssh/config ~/.ssh/id_ed25519
```

> There is no macOS Keychain here. GNOME's keyring agent usually unlocks keys at login;
> otherwise `AddKeysToAgent yes` prompts once per session.

Copy the public key:

```bash
# Wayland (Ubuntu 22.04+ default)
sudo apt install -y wl-clipboard && wl-copy < ~/.ssh/id_ed25519.pub

# X11
sudo apt install -y xclip && xclip -selection clipboard < ~/.ssh/id_ed25519.pub
```

Add it on GitHub: **Settings → SSH and GPG keys → New SSH key**.

**Verify:**

```bash
ssh -T git@github.com
# Expected: Hi username! You've successfully authenticated...
```



### Step 3.3 — Install GitHub CLI

From GitHub's official apt repo (fresher than Ubuntu's):

```bash
sudo mkdir -p -m 755 /etc/apt/keyrings
wget -qO- https://cli.github.com/packages/githubcli-archive-keyring.gpg \
  | sudo tee /etc/apt/keyrings/githubcli-archive-keyring.gpg > /dev/null
sudo chmod go+r /etc/apt/keyrings/githubcli-archive-keyring.gpg
echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/githubcli-archive-keyring.gpg] https://cli.github.com/packages stable main" \
  | sudo tee /etc/apt/sources.list.d/github-cli.list > /dev/null
sudo apt update && sudo apt install -y gh
```

```bash
gh auth login
```

Choose **GitHub.com → SSH → Login with browser**.

**Verify:**

```bash
gh auth status
gh repo list
```

---



## Phase 4 — GUI applications

Install these before or alongside CLI tools — you will use them throughout setup.

### Step 4.1 — Cursor (editor)

Cursor ships an AppImage for Linux (some releases also offer a `.deb`).

1. Download from [cursor.com](https://cursor.com) → **Download for Linux**.
2. Move it somewhere stable and make it executable:

```bash
mkdir -p ~/Applications
mv ~/Downloads/Cursor-*.AppImage ~/Applications/cursor.AppImage
chmod +x ~/Applications/cursor.AppImage
```

3. AppImages need FUSE 2:

```bash
sudo apt install -y libfuse2t64   # Ubuntu 24.04
# sudo apt install -y libfuse2    # Ubuntu 22.04
```

4. Open Cursor → sign in → install the shell command: **Ctrl+Shift+P → "Install 'cursor' command"**.

If that command is unavailable, add a wrapper yourself:

```bash
printf '#!/bin/sh\nexec "$HOME/Applications/cursor.AppImage" "$@"\n' > ~/.local/bin/cursor
chmod +x ~/.local/bin/cursor
```



### Step 4.2 — Google Chrome

```bash
wget https://dl.google.com/linux/direct/google-chrome-stable_current_amd64.deb
sudo apt install -y ./google-chrome-stable_current_amd64.deb
```

The `.deb` adds Google's apt repo, so Chrome updates with `apt upgrade`.

1. Sign in to sync bookmarks/extensions if desired.
2. Install the **Bitwarden** extension.



### Step 4.3 — Postman

```bash
sudo snap install postman
```

Or download the tarball from [postman.com](https://www.postman.com/downloads/) and extract it to `/opt`.



### Step 4.4 — Obsidian (notes / knowledge base)

Local Markdown notes with links and graph view — useful for docs, research, and personal wikis.

```bash
# .deb from https://obsidian.md/download
sudo apt install -y ./obsidian_*_amd64.deb
```

Alternatives: `sudo snap install obsidian --classic` or `flatpak install flathub md.obsidian.Obsidian`.

**Setup:**

1. Open **Obsidian**.
2. Create a new vault or open an existing folder.
3. (Optional) Sign in for Obsidian Sync / Publish if you use them.



### Step 4.5 — Local AI chat (Ollama + Open WebUI)

Atomic Chat is macOS / Apple Silicon only. The Linux equivalent is **Ollama** for the model runtime plus a GUI on top.

```bash
curl -fsSL https://ollama.com/install.sh | sh
ollama pull llama3.2
ollama run llama3.2
```

The installer detects NVIDIA/AMD GPUs and enables acceleration; CPU-only works but is slower.

**Add a GUI** — Open WebUI runs in Docker (needs Phase 6):

```bash
docker run -d -p 3000:8080 --add-host=host.docker.internal:host-gateway \
  -v open-webui:/app/backend/data --name open-webui --restart always \
  ghcr.io/open-webui/open-webui:main
# then open http://localhost:3000
```

Desktop-app alternatives: [Jan](https://jan.ai/), [GPT4All](https://www.nomic.ai/gpt4all), [LM Studio](https://lmstudio.ai/).

> First model download can be multi-GB. Larger models need more RAM/VRAM (rough guide: ~8 GB for 3B, ~16 GB for 7B, ~32 GB for 13B).



### Step 4.6 — Terax (AI-native terminal)

[Terax](https://github.com/crynta/terax-ai) is a ~8 MB terminal-based development environment: native PTY backend with WebGL rendering, an agentic AI side panel, editor, file explorer, git visualization, and web preview. No telemetry, no account required.

Download the latest build from [Releases](https://github.com/crynta/terax-ai/releases/latest) — `.deb` and `.AppImage` are the Ubuntu-relevant assets.

**`.deb` (recommended on Ubuntu):**

```bash
sudo apt install -y ./Terax_*_amd64.deb
```

**AppImage:**

```bash
mkdir -p ~/Applications
mv ~/Downloads/Terax_*.AppImage ~/Applications/terax.AppImage
chmod +x ~/Applications/terax.AppImage
~/Applications/terax.AppImage
```

Needs FUSE 2 — already installed if you did Step 4.1, otherwise `sudo apt install -y libfuse2t64` (24.04) or `libfuse2` (22.04).

**Other distros:** `yay -S terax-bin` (Arch/AUR), `nix profile install github:crynta/terax-ai` (NixOS), or the `.rpm` for Fedora/openSUSE.

**Configure an AI provider:**

Go to **Settings → AI**, pick a provider (OpenAI, Anthropic, Google, or a local model via Ollama), and paste your API key. Keys are stored in the OS keychain.

> Prefer local models? Point Terax at the **Ollama** instance from Step 4.5 — no API key needed.

**Build from source (optional)** — needs Rust (Phase 5.5), Node 20+ (Phase 5.1), pnpm, and the Tauri Linux prerequisites:

```bash
sudo apt install -y libwebkit2gtk-4.1-dev libgtk-3-dev libayatana-appindicator3-dev librsvg2-dev
git clone https://github.com/crynta/terax-ai.git
cd terax-ai
pnpm install
pnpm tauri build
```

---



## Phase 5 — Language runtimes

Install all five runtimes. Order does not matter much, but do each verify step before moving on.

### Step 5.1 — Node.js (via nvm)

```bash
curl -o- https://raw.githubusercontent.com/nvm-sh/nvm/v0.40.1/install.sh | bash
```

Ensure the nvm lines are in `~/.zshrc` (see Step 2.6), then:

```bash
source ~/.zshrc
nvm install --lts
nvm alias default 'lts/*'
nvm use default
```

**Verify:**

```bash
node --version    # e.g. v22.x
npm --version
```

**Useful globals (optional):**

```bash
npm install -g typescript tsx prettier eslint
```

> Do **not** `apt install nodejs` alongside nvm — the two will fight over `PATH`.



### Step 5.2 — Bun

```bash
sudo apt install -y unzip
curl -fsSL https://bun.sh/install | bash
source ~/.zshrc
```

**Verify:**

```bash
bun --version
```



### Step 5.3 — Python (via pyenv)

pyenv builds Python from source, so install the build dependencies first:

```bash
sudo apt install -y make build-essential libssl-dev zlib1g-dev libbz2-dev \
  libreadline-dev libsqlite3-dev wget curl llvm libncursesw5-dev xz-utils \
  tk-dev libxml2-dev libxmlsec1-dev libffi-dev liblzma-dev
```

```bash
curl -fsSL https://pyenv.run | bash
```

Add the pyenv init lines to `~/.zshrc` (see Step 2.6), then:

```bash
source ~/.zshrc
pyenv install 3.12.7
pyenv global 3.12.7
```

**Verify:**

```bash
python --version   # Python 3.12.x
pip --version
```

**Useful tools:**

```bash
pip install pipx
pipx ensurepath
pipx install poetry ruff
```

> Ubuntu's system Python is used by the OS itself. Never `pip install` into it —
> that is what pyenv and `pipx` are for.



### Step 5.4 — Go

Ubuntu's `golang` package lags behind. Install the official tarball:

```bash
GO_VERSION=1.23.4    # check https://go.dev/dl/ for the current release
wget "https://go.dev/dl/go${GO_VERSION}.linux-amd64.tar.gz"
sudo rm -rf /usr/local/go
sudo tar -C /usr/local -xzf "go${GO_VERSION}.linux-amd64.tar.gz"
```

Ensure `GOPATH` and `/usr/local/go/bin` are in `~/.zshrc` (see Step 2.6), then:

```bash
source ~/.zshrc
go install golang.org/x/tools/gopls@latest
go install github.com/go-delve/delve/cmd/dlv@latest
```

**Verify:**

```bash
go version
gopls version
```



### Step 5.5 — Rust

```bash
curl --proto '=https' --tlsv1.2 -sSf https://sh.rustup.rs | sh
```

Choose the default install (option 1). Then:

```bash
source "$HOME/.cargo/env"
rustup default stable
rustup component add rustfmt clippy
```

**Verify:**

```bash
rustc --version
cargo --version
```

---



## Phase 6 — Containers (Docker Engine)

No Colima or VM needed — Linux runs containers natively. Install Docker Engine from Docker's official repo, not Ubuntu's `docker.io` package.

### Step 6.1 — Add Docker's repo and install

```bash
sudo install -m 0755 -d /etc/apt/keyrings
sudo curl -fsSL https://download.docker.com/linux/ubuntu/gpg -o /etc/apt/keyrings/docker.asc
sudo chmod a+r /etc/apt/keyrings/docker.asc

echo "deb [arch=$(dpkg --print-architecture) signed-by=/etc/apt/keyrings/docker.asc] https://download.docker.com/linux/ubuntu $(. /etc/os-release && echo "$VERSION_CODENAME") stable" \
  | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null

sudo apt update
sudo apt install -y docker-ce docker-ce-cli containerd.io \
  docker-buildx-plugin docker-compose-plugin
```



### Step 6.2 — Run Docker without sudo

```bash
sudo usermod -aG docker "$USER"
newgrp docker
```

Log out and back in so the group membership applies everywhere.

> Adding your user to the `docker` group grants root-equivalent access to the host.
> On a shared or hardened machine, use [rootless mode](https://docs.docker.com/engine/security/rootless/) instead.

### Step 6.3 — Verify Docker works

```bash
docker run hello-world
docker ps -a
docker compose version
```



### Step 6.4 — Auto-start

Docker installs a systemd unit and starts on boot by default:

```bash
sudo systemctl enable --now docker
systemctl status docker
```

---



## Phase 7 — DevOps CLI



### Step 7.1 — kubectl

```bash
K8S_MINOR=v1.33   # check https://kubernetes.io/releases/ and use the current minor
sudo mkdir -p -m 755 /etc/apt/keyrings
curl -fsSL "https://pkgs.k8s.io/core:/stable:/${K8S_MINOR}/deb/Release.key" \
  | sudo gpg --dearmor -o /etc/apt/keyrings/kubernetes-apt-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/kubernetes-apt-keyring.gpg] https://pkgs.k8s.io/core:/stable:/${K8S_MINOR}/deb/ /" \
  | sudo tee /etc/apt/sources.list.d/kubernetes.list > /dev/null
sudo apt update && sudo apt install -y kubectl
kubectl version --client
```

If you use a specific cluster manager:

```bash
# Optional — pick what you need
sudo apt install -y kubectx        # fast context switching (also provides kubens)
sudo snap install helm --classic   # Kubernetes package manager
```



### Step 7.2 — Terraform

```bash
wget -O- https://apt.releases.hashicorp.com/gpg \
  | sudo gpg --dearmor -o /etc/apt/keyrings/hashicorp-archive-keyring.gpg
echo "deb [signed-by=/etc/apt/keyrings/hashicorp-archive-keyring.gpg] https://apt.releases.hashicorp.com $(. /etc/os-release && echo "$VERSION_CODENAME") main" \
  | sudo tee /etc/apt/sources.list.d/hashicorp.list > /dev/null
sudo apt update && sudo apt install -y terraform
terraform version
```

**Optional — Terraform version manager:**

```bash
git clone --depth=1 https://github.com/tfutils/tfenv.git ~/.tfenv
ln -sf ~/.tfenv/bin/* ~/.local/bin
tfenv install latest
tfenv use latest
```

---



## Phase 8 — Recommended extras

These are not required but commonly useful for day-to-day engineering.

```bash
# Search and navigation
sudo apt install -y ripgrep bat jq eza

# Network and API debugging
sudo apt install -y httpie curl wget net-tools dnsutils

# Database clients (CLI)
sudo apt install -y postgresql-client redis-tools

# Version control extras
sudo apt install -y git-delta
sudo snap install lazygit

# Watching files, process inspection
sudo apt install -y entr htop tree
```

Ubuntu installs bat as **`batcat`**. Alias it in `~/.zshrc`:

```bash
alias bat="batcat"
alias fd="fdfind"   # only if you skipped the symlink in Step 2.5
```

> On Ubuntu 22.04, `eza` is not in the repos — install via
> [the eza apt repo](https://github.com/eza-community/eza/blob/main/INSTALL.md) or `cargo install eza`.
>
> `yq` is not packaged either: `sudo snap install yq`.

---



## Phase 9 — Cursor agent skills (optional)

If you use Cursor agent skills for AI-assisted workflows:

```bash
npx skills add https://github.com/mohi-devhub/antivibe --skill antivibe
npx skills add https://github.com/juliusbrussee/caveman --skill caveman
npx skills add https://github.com/juliusbrussee/caveman --skill caveman-commit
npx skills add https://github.com/anthropics/skills --skill doc-coauthoring
npx skills add https://github.com/anthropics/skills --skill frontend-design
npx skills add https://github.com/vercel-labs/agent-skills --skill web-design-guidelines
npx skills add https://github.com/mattpocock/skills --skill improve-codebase-architecture
```

Browse more at [skills.sh](https://skills.sh).

---



## Phase 10 — CodeGraph (optional)

[CodeGraph](https://github.com/colbymchenry/codegraph) is a local-first code knowledge graph that wires into Cursor (and other agents) over MCP. The index stays on your machine; agents use the graph instead of grepping the whole repo on every question.

Requires Cursor (Phase 4). The standalone installer bundles its own runtime — Node is not required.

### Step 10.1 — Install the CLI

```bash
curl -fsSL https://raw.githubusercontent.com/colbymchenry/codegraph/main/install.sh | sh
```

The installer puts `codegraph` in `~/.local/bin`. Open a **new terminal** (or `source ~/.zshrc`) so the command resolves.

If `~/.local/bin` is not on your PATH, add this to `~/.zshrc`:

```bash
export PATH="$HOME/.local/bin:$PATH"
```

**Already have Node?** You can use npm instead:

```bash
npm i -g @colbymchenry/codegraph
```

**Verify:**

```bash
codegraph --version
```



### Step 10.2 — Wire up your agent(s)

```bash
codegraph install
```

This detects installed agents (Cursor, Claude Code, Codex CLI, and others) and writes the CodeGraph MCP config. Choose **global** so it applies to all projects, or **local** for the current repo only.

Non-interactive shortcut (auto-detect agents, global):

```bash
codegraph install --yes
```

Or target Cursor explicitly:

```bash
codegraph install --target=cursor --yes
```

**Restart Cursor** (fully quit and reopen) so the MCP server loads.



### Step 10.3 — Initialize each project

Run once per repo you want indexed:

```bash
cd your-project
codegraph init
```

Creates `.codegraph/` and builds the graph. Auto-sync is on by default — the index updates as files change.

**Verify:**

```bash
codegraph status
```



### Step 10.4 — Upgrade / uninstall

```bash
codegraph upgrade          # update in place
codegraph upgrade --check  # see if an update is available
codegraph uninstall        # remove agent configs + CLI
codegraph uninit           # remove this project's .codegraph/ index only
```

Docs: [codegraph installation](https://colbymchenry.github.io/codegraph/getting-started/installation/).

---



## Phase 11 — Context7 (optional)

[Context7](https://github.com/upstash/context7) pulls up-to-date, version-specific library documentation and code examples into Cursor's context, instead of relying on outdated training data. Setup offers two modes: **MCP** (Cursor calls Context7 tools natively) or **CLI + Skills** (installs a skill that runs `ctx7` commands).

Requires Node.js 18+ (Phase 5) and Cursor (Phase 4).

### Step 11.1 — Get an API key (recommended)

Free at [context7.com/dashboard](https://context7.com/dashboard) — raises your rate limit. Setup works without one via OAuth login, just with a lower quota.

### Step 11.2 — Run setup for Cursor

```bash
npx ctx7 setup --cursor
```

Prompts for OAuth login, then lets you pick MCP or CLI + Skills mode. Non-interactive shortcuts:

```bash
npx ctx7 setup --cursor --mcp --yes    # MCP mode (native tools)
npx ctx7 setup --cli --cursor --yes    # CLI + Skills mode only
```

Already have a key from Step 11.1? Pass it directly:

```bash
npx ctx7 setup --cursor --api-key YOUR_API_KEY
```

**Restart Cursor** (fully quit and reopen) so the MCP server loads, if you chose MCP mode.

### Step 11.3 — Optional global CLI

```bash
npm install -g ctx7
```

Skip this and keep using `npx ctx7@latest ...` if you'd rather not install it globally.

**Verify:**

```bash
npx ctx7 --version
```

### Step 11.4 — Optional rule

`ctx7 setup` already installs a skill that triggers Context7 automatically. If you prefer an explicit rule instead, add this in **Cursor Settings → Rules and Commands**:

```
Always use Context7 when I need library/API documentation, code generation,
setup or configuration steps without me having to explicitly ask.
```

### Step 11.5 — Remove

```bash
npx ctx7 remove --cursor
npm uninstall -g ctx7   # only if you installed the CLI globally in Step 11.3
```

Docs: [Context7 CLI reference](https://context7.com/docs/clients/cli).

---



## Final verification checklist

Run through this list. Every item should pass before you call setup done.

```bash
# Phase 0
gcc --version && git --version

# Phase 2
echo $SHELL          # /usr/bin/zsh
fd --version

# Phase 3
git config user.email
ssh -T git@github.com
gh auth status

# Phase 5 — languages
node --version && npm --version
bun --version
python --version && pip --version
go version
rustc --version && cargo --version

# Phase 6
systemctl is-active docker
docker run --rm hello-world
docker compose version

# Phase 7
kubectl version --client
terraform version

# Phase 10 (optional)
codegraph --version

# Phase 11 (optional)
npx ctx7 --version
```

**GUI sanity check:**

- [ ] Bitwarden unlocks and autofills
- [ ] VPN client connects (`curl https://ipinfo.io/ip` shows the tunnel IP)
- [ ] Cursor opens and `cursor .` works in terminal
- [ ] Chrome + Bitwarden extension work
- [ ] Postman launches
- [ ] Obsidian opens and a vault loads
- [ ] `ollama run llama3.2` responds (Open WebUI loads at localhost:3000)
- [ ] Terax opens and an AI provider is configured (Settings → AI)
- [ ] CodeGraph MCP shows in Cursor (after `codegraph install` + restart)
- [ ] Context7 MCP shows in Cursor (after `ctx7 setup --cursor` + restart, if MCP mode)

---



## Troubleshooting



### zsh is installed but bash still loads

```bash
chsh -s "$(which zsh)"
```

Then **log out and back in** — `chsh` only applies to new login sessions. Verify with `echo $SHELL`.

### Powerlevel10k shows broken icons

Install MesloLGS NF (Step 2.4), select it in your terminal profile's font setting, then run `p10k configure`.

### fd: command not found

Ubuntu names the binary `fdfind`:

```bash
ln -sf "$(which fdfind)" ~/.local/bin/fd
source ~/.zshrc
```

Same story for `bat` → `batcat`.

### nvm: command not found

Ensure the nvm block is in `~/.zshrc` and run `source ~/.zshrc`. Check:

```bash
ls ~/.nvm/nvm.sh
```

### pyenv: BUILD FAILED / missing headers

Install the full build dependency list from Step 5.3, then retry `pyenv install`.

### pyenv: python still shows the system version

```bash
pyenv global 3.12.7
pyenv rehash
which python   # should be ~/.pyenv/shims/python
```

### Docker: permission denied on /var/run/docker.sock

```bash
sudo usermod -aG docker "$USER"
newgrp docker          # current shell only
```

Log out and back in for a permanent fix. Confirm with `id -nG | grep docker`.

### Docker: Cannot connect to the Docker daemon

```bash
sudo systemctl start docker
sudo systemctl enable docker
systemctl status docker
```

### AppImage will not launch (Cursor)

```bash
sudo apt install -y libfuse2t64   # 24.04
# sudo apt install -y libfuse2    # 22.04
chmod +x ~/Applications/cursor.AppImage
```

If it still fails, run it with `--appimage-extract-and-run` to see the real error.

### GitHub SSH: Permission denied (publickey)

```bash
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
ssh -T git@github.com
```

Confirm the public key is added on GitHub, `~/.ssh` is `700`, and keys are `600`.

### apt: NO_PUBKEY or repository signature errors

Re-run the keyring step for that repo (Phases 3, 6, 7). Keys live in `/etc/apt/keyrings/`, and each `sources.list.d` entry must reference the matching `signed-by=` path.

### codegraph: command not found

Open a new terminal, or ensure `~/.local/bin` is on your PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
source ~/.zshrc
codegraph --version
```

If you installed via npm instead: `npm i -g @colbymchenry/codegraph`.

### Context7: quota exceeded or MCP not showing up

```bash
npx ctx7 login          # re-authenticate for higher rate limits
```

If Cursor does not list the Context7 MCP server, fully quit and reopen Cursor after running `ctx7 setup --cursor`. If `ctx7` is not found, use `npx ctx7@latest` or install it globally: `npm install -g ctx7`.

---



## Maintenance

Run periodically to keep tools current:

```bash
sudo apt update && sudo apt upgrade -y && sudo apt autoremove -y
sudo snap refresh
npm update -g
rustup update
go install golang.org/x/tools/gopls@latest
docker system prune -f          # reclaim image/container space
codegraph upgrade               # if CodeGraph is installed
brew update && brew upgrade     # if Homebrew for Linux is installed
```

Reboot after kernel upgrades — `ls /var/run/reboot-required` tells you when one is pending.

---



## Quick reference — install order

```
Phase 0   Bitwarden → VPN client → GitHub account → build-essential
Phase 1   apt (built in) → Homebrew for Linux (optional)
Phase 2   zsh + chsh → Oh My Zsh → plugins → Powerlevel10k → fzf/fd → ~/.zshrc
Phase 3   Git config → SSH key → gh CLI
Phase 4   Cursor → Chrome → Postman → Obsidian → Ollama + Open WebUI → Terax
Phase 5   nvm/Node → Bun → pyenv/Python → Go → Rust
Phase 6   Docker Engine + Compose plugin → docker group
Phase 7   kubectl → Terraform
Phase 8   Optional CLI extras
Phase 9   Optional Cursor skills
Phase 10  Optional CodeGraph (CLI → install → init per project)
Phase 11  Optional Context7 (npx ctx7 setup --cursor)
Verify  Run checklist
```

---



## Differences from the macOS guide

| macOS ([Mac.md](Mac.md))       | Ubuntu (this guide)                          |
| ------------------------------ | -------------------------------------------- |
| Xcode + Command Line Tools     | `build-essential` + dev headers              |
| Homebrew (required)            | `apt` (Homebrew optional)                    |
| zsh is the default shell       | bash is default — install zsh and `chsh`     |
| Colima VM for Docker           | Docker Engine runs natively                  |
| `pbcopy`                       | `wl-copy` (Wayland) / `xclip` (X11)          |
| `ssh-add --apple-use-keychain` | `ssh-add` + `AddKeysToAgent yes`             |
| Shadowrocket                   | Clash Verge Rev / sing-box / v2rayA          |
| Atomic Chat                    | Ollama + Open WebUI (or Jan / GPT4All)       |
| `brew install fd bat`          | `fdfind` / `batcat` — symlink or alias them  |

---



## License

This guide is free to copy and adapt. Tool licenses belong to their respective vendors.
