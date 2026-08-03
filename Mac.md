# Mac Setup for Software Engineering

Step-by-step guide to take a fresh Mac from zero to a fully featured software engineering machine.

Works on Apple Silicon (M-series) and Intel Macs. Commands assume **zsh** (macOS default).

---

## What you get


| Category          | Tools                                              |
| ----------------- | -------------------------------------------------- |
| Security & access | Bitwarden, Shadowrocket, GitHub                    |
| Apple dev base    | Xcode, Command Line Tools                          |
| Package manager   | Homebrew                                           |
| Shell             | zsh, Oh My Zsh, Powerlevel10k, fzf, fd             |
| Languages         | Node (nvm), Bun, Python (pyenv), Go, Rust          |
| Containers        | Colima + Docker CLI                                |
| DevOps CLI        | git, GitHub CLI, kubectl, Terraform                |
| GUI apps          | Cursor, Chrome, Postman, Obsidian, Atomic Chat, Terax |
| AI code intel     | CodeGraph (local MCP knowledge graph for agents)   |


---



## Before you start

- Connect to Wi-Fi.
- Sign in with your Apple ID (System Settings → Apple Account).
- Plug in power — some steps take a while.

Estimated time: **2–4 Minutes**.

---



## Phase 0 — Essentials (before dev tools)

Do these first. You need passwords, network access, and Apple dev tooling before everything else.

### Step 0.1 — Bitwarden (password manager)

1. Open **App Store** → search **Bitwarden** → Install.
2. Open Bitwarden → sign in.
3. Install the **Safari** or **Chrome** browser extension when prompted.
4. You now have your PASSWORDS!



### Step 0.2 — Shadowrocket (VPN)

1. Open **App Store** → search **Shadowrocket** → Install.
2. Import your VPN subscription.
3. Connect and verify browsing works.

> Use VPN before signing into sensitive accounts.



### Step 0.3 — Sign in to GitHub

Go to github.com → Use Bitwarden → Sign in.

You will configure SSH keys and `gh` CLI later in Phase 4.

### Step 0.4 — Xcode and Command Line Tools

Xcode is required for native compilation, iOS/macOS work, and many CLI tools.

**Install Xcode:**

1. Open **App Store** → search **Xcode** → Install (~12 GB).
2. Open Xcode once → accept the license agreement.
3. Wait for additional components to finish installing.

**Install Command Line Tools (if not already present):**

```bash
xcode-select --install
```

Click **Install** in the dialog. If you already installed Xcode, CLT may already be present.

**Accept the license (required for some tools):**

```bash
sudo xcodebuild -license accept
```

**Verify:**

```bash
xcode-select -p
# Expected: /Applications/Xcode.app/Contents/Developer
# or: /Library/Developer/CommandLineTools

git --version
# Expected: git version 2.x
```

---



## Phase 1 — Homebrew

Homebrew is the package manager for everything that follows.

### Step 1.1 — Install Homebrew

```bash
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"
```

Follow the on-screen instructions. On Apple Silicon, Homebrew installs to `/opt/homebrew`.

### Step 1.2 — Add Homebrew to your shell

The installer prints these commands — run them (example):

```bash
echo >> ~/.zprofile
echo 'eval "$(/opt/homebrew/bin/brew shellenv zsh)"' >> ~/.zprofile
eval "$(/opt/homebrew/bin/brew shellenv zsh)"
```

**Verify:**

```bash
brew --version
brew doctor
```

Fix anything `brew doctor` flags before continuing.

---



## Phase 2 — Shell setup

A good shell saves hours every week.

### Step 2.1 — Install Oh My Zsh

```bash
sh -c "$(curl -fsSL https://raw.githubusercontent.com/ohmyzsh/ohmyzsh/master/tools/install.sh)"
```

Choose **zsh** as your default shell when asked.

### Step 2.2 — Install zsh plugins

```bash
git clone https://github.com/zsh-users/zsh-autosuggestions \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-autosuggestions

git clone https://github.com/zsh-users/zsh-syntax-highlighting \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/plugins/zsh-syntax-highlighting
```



### Step 2.3 — Install Powerlevel10k theme

```bash
git clone --depth=1 https://github.com/romkatv/powerlevel10k.git \
  ${ZSH_CUSTOM:-~/.oh-my-zsh/custom}/themes/powerlevel10k
```



### Step 2.4 — Install fzf and fd

```bash
brew install fzf fd
$(brew --prefix)/opt/fzf/install
```

Press **y** to enable fuzzy history search (Ctrl+R) and key bindings.

### Step 2.5 — Configure ~/.zshrc

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
# Bun
export BUN_INSTALL="$HOME/.bun"
export PATH="$BUN_INSTALL/bin:$PATH"

# Go
export GOPATH="$HOME/go"
export PATH="$PATH:$GOPATH/bin"

# pyenv (after Python install)
export PYENV_ROOT="$HOME/.pyenv"
export PATH="$PYENV_ROOT/bin:$PATH"
eval "$(pyenv init - zsh)"

# nvm (after Node install — see Step 5.1)
export NVM_DIR="$HOME/.nvm"
[ -s "$HOMEBREW_PREFIX/opt/nvm/nvm.sh" ] && \. "$HOMEBREW_PREFIX/opt/nvm/nvm.sh"
[ -s "$HOMEBREW_PREFIX/opt/nvm/etc/bash_completion.d/nvm" ] && \
  \. "$HOMEBREW_PREFIX/opt/nvm/etc/bash_completion.d/nvm"

# fzf
[ -f ~/.fzf.zsh ] && source ~/.fzf.zsh
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
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
```

Add to `~/.ssh/config`:

```
Host github.com
  AddKeysToAgent yes
  UseKeychain yes
  IdentityFile ~/.ssh/id_ed25519
```

Copy the public key:

```bash
pbcopy < ~/.ssh/id_ed25519.pub
```

Add it on GitHub: **Settings → SSH and GPG keys → New SSH key**.

**Verify:**

```bash
ssh -T git@github.com
# Expected: Hi username! You've successfully authenticated...
```



### Step 3.3 — Install GitHub CLI

```bash
brew install gh
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

1. Download from [cursor.com](https://cursor.com).
2. Drag **Cursor** to **Applications**.
3. Open Cursor → sign in → install shell command: **Cmd+Shift+P → "Install 'cursor' command"**.



### Step 4.2 — Google Chrome

1. Download from [google.com/chrome](https://www.google.com/chrome/).
2. Install and sign in to sync bookmarks/extensions if desired.
3. Install the **Bitwarden** extension.



### Step 4.3 — Postman

```bash
brew install --cask postman
```

Or download from [postman.com](https://www.postman.com/downloads/).



### Step 4.4 — Obsidian (notes / knowledge base)

Local Markdown notes with links and graph view — useful for docs, research, and personal wikis.

```bash
brew install --cask obsidian
```

Or download from [obsidian.md/download](https://obsidian.md/download).

**Setup:**

1. Open **Obsidian**.
2. Create a new vault or open an existing folder.
3. (Optional) Sign in for Obsidian Sync / Publish if you use them.



### Step 4.5 — Atomic Chat (local AI chat)

[Atomic Chat](https://atomic.chat/) runs open-source AI models fully on-device — no cloud, no subscription. Requires **macOS 13+** and **Apple Silicon** (M1 or newer).

1. Download from [atomic.chat](https://atomic.chat/) → **Download for Mac**.
2. Open the `.dmg` (or unzip), drag **Atomic Chat** to **Applications**.
3. Open the app → pick a model (Llama, Qwen, DeepSeek, etc.) → download → start chatting.

Or install from the **Mac App Store** (search **Atomic Chat**).

> First model download can be multi-GB. Prefer Wi-Fi and plugged-in power. Larger models need more RAM (rough guide: ~8 GB for 3B, ~16 GB for 7B, ~32 GB for 13B).



### Step 4.6 — Terax (AI-native terminal)

[Terax](https://github.com/crynta/terax-ai) is a ~8 MB terminal-based development environment: native PTY backend with WebGL rendering, an agentic AI side panel, editor, file explorer, git visualization, and web preview. No telemetry, no account required. Works on Intel and Apple Silicon.

1. Download the latest `.dmg` from [Releases](https://github.com/crynta/terax-ai/releases/latest).
2. Open the `.dmg` → drag **Terax** to **Applications**.
3. Open Terax. The app auto-updates from then on.

**Configure an AI provider:**

Go to **Settings → AI**, pick a provider (OpenAI, Anthropic, Google, or a local model via Ollama), and paste your API key. Keys are stored in the macOS Keychain.

> Prefer local models? Point Terax at **Atomic Chat** or Ollama instead of a cloud provider — no API key needed.

**Build from source (optional)** — needs Rust (Phase 5.5), Node 20+ (Phase 5.1), and pnpm:

```bash
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
brew install nvm
mkdir -p ~/.nvm
```

Ensure nvm lines are in `~/.zshrc` (see Step 2.5), then:

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



### Step 5.2 — Bun

```bash
curl -fsSL https://bun.sh/install | bash
source ~/.zshrc
```

**Verify:**

```bash
bun --version
```



### Step 5.3 — Python (via pyenv)

```bash
brew install pyenv pyenv-virtualenv
```

Add pyenv init to `~/.zshrc` (see Step 2.5), then:

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



### Step 5.4 — Go

```bash
brew install go
```

Ensure `GOPATH` is in `~/.zshrc` (see Step 2.5), then:

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

Choose default install (option 1). Then:

```bash
source ~/.zshrc   # or: source "$HOME/.cargo/env"
rustup default stable
rustup component add rustfmt clippy
```

**Verify:**

```bash
rustc --version
cargo --version
```

---



## Phase 6 — Containers (Colima)

Colima runs Docker-compatible containers without Docker Desktop.

### Step 6.1 — Install Colima and Docker CLI

```bash
brew install colima docker docker-compose
```



### Step 6.2 — Start Colima

```bash
colima start --cpu 4 --memory 8 --disk 60
```

Adjust CPU/RAM to your Mac. Apple Silicon uses `--vm-type vz` automatically on recent Colima versions.

### Step 6.3 — Verify Docker works

```bash
docker run hello-world
docker ps -a
```



### Step 6.4 — Auto-start (optional)

Colima does not auto-start on boot by default. Add an alias or start manually:

```bash
colima start
```

Or create a LaunchAgent — see [Colima docs](https://github.com/abiosoft/colima).

---



## Phase 7 — DevOps CLI



### Step 7.1 — kubectl

```bash
brew install kubectl
kubectl version --client
```

If you use a specific cluster manager:

```bash
# Optional — pick what you need
brew install kubectx   # fast context switching
brew install helm      # Kubernetes package manager
```



### Step 7.2 — Terraform

```bash
brew tap hashicorp/tap
brew install hashicorp/tap/terraform
terraform version
```

**Optional — Terraform version manager:**

```bash
brew install tfenv
tfenv install latest
tfenv use latest
```

---



## Phase 8 — Recommended extras

These are not required but commonly useful for day-to-day engineering.

```bash
# Search and navigation
brew install ripgrep bat eza jq yq

# Network and API debugging
brew install httpie curl wget

# Database clients (CLI)
brew install postgresql redis

# Version control extras
brew install lazygit delta

# JSON/YAML processing, watching files, etc.
brew install watch entr
```

Add aliases in `~/.zshrc` as you prefer.

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

[CodeGraph](https://github.com/colbymchenry/codegraph) is a local-first code knowledge graph that wires into Cursor (and other agents) over MCP. Index stays on your machine; agents use the graph instead of grepping the whole repo on every question.

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



## Final verification checklist

Run through this list. Every item should pass before you call setup done.

```bash
# Phase 0
xcode-select -p && git --version

# Phase 1
brew --version

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
colima status
docker run --rm hello-world

# Phase 7
kubectl version --client
terraform version

# Phase 10 (optional)
codegraph --version
```

**GUI sanity check:**

- [ ] Bitwarden unlocks and autofills
- [ ] Shadowrocket connects
- [ ] Cursor opens and `cursor .` works in terminal
- [ ] Chrome + Bitwarden extension work
- [ ] Postman launches
- [ ] Obsidian opens and a vault loads
- [ ] Atomic Chat opens and a model can be downloaded (Apple Silicon)
- [ ] Terax opens and an AI provider is configured (Settings → AI)
- [ ] CodeGraph MCP shows in Cursor (after `codegraph install` + restart)

---



## Troubleshooting



### Homebrew command not found after install

```bash
eval "$(/opt/homebrew/bin/brew shellenv zsh)"   # Apple Silicon
# eval "$(/usr/local/bin/brew shellenv zsh)"    # Intel
```

Add the `eval` line permanently to `~/.zprofile`.

### nvm: command not found

Ensure Homebrew nvm paths are in `~/.zshrc` and run `source ~/.zshrc`. Check:

```bash
brew --prefix nvm
ls "$(brew --prefix nvm)/nvm.sh"
```



### pyenv: python still shows system version

```bash
pyenv global 3.12.7
pyenv rehash
which python   # should be ~/.pyenv/shims/python
```



### Docker: Cannot connect to the Docker daemon

```bash
colima status
colima start
docker context ls   # should show colima as current
```



### GitHub SSH: Permission denied (publickey)

```bash
ssh-add --apple-use-keychain ~/.ssh/id_ed25519
ssh -T git@github.com
```

Confirm the public key is added on GitHub.

### Xcode license not accepted

```bash
sudo xcodebuild -license accept
```

### codegraph: command not found

Open a new terminal, or ensure `~/.local/bin` is on your PATH:

```bash
export PATH="$HOME/.local/bin:$PATH"
source ~/.zshrc
codegraph --version
```

If you installed via npm instead: `npm i -g @colbymchenry/codegraph`.

---



## Maintenance

Run periodically to keep tools current:

```bash
brew update && brew upgrade && brew cleanup
npm update -g
rustup update
go install golang.org/x/tools/gopls@latest
colima stop && colima start   # after Colima upgrades
codegraph upgrade             # if CodeGraph is installed
```

---



## Quick reference — install order

```
Phase 0   Bitwarden → Shadowrocket → GitHub account → Xcode + CLT
Phase 1   Homebrew
Phase 2   Oh My Zsh → plugins → Powerlevel10k → fzf/fd → ~/.zshrc
Phase 3   Git config → SSH key → gh CLI
Phase 4   Cursor → Chrome → Postman → Obsidian → Atomic Chat → Terax
Phase 5   nvm/Node → Bun → pyenv/Python → Go → Rust
Phase 6   Colima + Docker
Phase 7   kubectl → Terraform
Phase 8   Optional CLI extras
Phase 9   Optional Cursor skills
Phase 10  Optional CodeGraph (CLI → install → init per project)
Verify  Run checklist
```

---



## License

This guide is free to copy and adapt. Tool licenses belong to their respective vendors.
