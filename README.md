# setup-my-machine

Opinionated, copy-paste setup guides that take a **fresh machine from zero to a fully featured software engineering environment** — password manager, VPN, shell, language runtimes, containers, DevOps CLIs, editors, and local AI tooling.

Both guides follow the same phased structure, so you can start at Phase 0 on a brand-new laptop and work straight through, or jump to a single phase when you only need one piece.

---

## Guides

| Guide                  | Platform                                  |
| ---------------------- | ----------------------------------------- |
| **[Mac.md](Mac.md)**   | macOS — Apple Silicon (M-series) and Intel |
| **[Linux.md](Linux.md)** | Ubuntu 24.04 LTS (22.04 LTS notes included) |

---

## Table of contents

### [Mac.md](Mac.md) — macOS

| Phase | Contents |
| ----- | -------- |
| [Before you start](Mac.md#before-you-start) | Wi-Fi, Apple ID, power |
| [Phase 0 — Essentials](Mac.md#phase-0--essentials-before-dev-tools) | Bitwarden, Shadowrocket, GitHub, Xcode + Command Line Tools |
| [Phase 1 — Homebrew](Mac.md#phase-1--homebrew) | Install and wire into the shell |
| [Phase 2 — Shell setup](Mac.md#phase-2--shell-setup) | Oh My Zsh, plugins, Powerlevel10k, fzf/fd, `~/.zshrc` |
| [Phase 3 — Git and GitHub CLI](Mac.md#phase-3--git-and-github-cli) | Git identity, SSH key, `gh` |
| [Phase 4 — GUI applications](Mac.md#phase-4--gui-applications) | Cursor, Chrome, Postman, Obsidian, Atomic Chat, Terax |
| [Phase 5 — Language runtimes](Mac.md#phase-5--language-runtimes) | Node (nvm), Bun, Python (pyenv), Go, Rust |
| [Phase 6 — Containers](Mac.md#phase-6--containers-colima) | Colima + Docker CLI |
| [Phase 7 — DevOps CLI](Mac.md#phase-7--devops-cli) | kubectl, Terraform |
| [Phase 8 — Recommended extras](Mac.md#phase-8--recommended-extras) | ripgrep, bat, eza, jq, lazygit, … |
| [Phase 9 — Cursor agent skills](Mac.md#phase-9--cursor-agent-skills-optional) | Optional AI workflow skills |
| [Phase 10 — CodeGraph](Mac.md#phase-10--codegraph-optional) | Optional local code knowledge graph (MCP) |
| [Phase 11 — Context7](Mac.md#phase-11--context7-optional) | Optional up-to-date library docs for agents (MCP) |
| [Verification](Mac.md#final-verification-checklist) · [Troubleshooting](Mac.md#troubleshooting) · [Maintenance](Mac.md#maintenance) | Checks, fixes, upkeep |

### [Linux.md](Linux.md) — Ubuntu

| Phase | Contents |
| ----- | -------- |
| [Before you start](Linux.md#before-you-start) | Network, `sudo` group, system update |
| [Phase 0 — Essentials](Linux.md#phase-0--essentials-before-dev-tools) | Bitwarden, VPN client, GitHub, `build-essential` |
| [Phase 1 — Package managers](Linux.md#phase-1--package-managers) | apt, optional Homebrew for Linux |
| [Phase 2 — Shell setup](Linux.md#phase-2--shell-setup) | zsh + `chsh`, Oh My Zsh, Powerlevel10k, fzf/fd, `~/.zshrc` |
| [Phase 3 — Git and GitHub CLI](Linux.md#phase-3--git-and-github-cli) | Git identity, SSH key, `gh` |
| [Phase 4 — GUI applications](Linux.md#phase-4--gui-applications) | Cursor, Chrome, Postman, Obsidian, Ollama + Open WebUI, Terax |
| [Phase 5 — Language runtimes](Linux.md#phase-5--language-runtimes) | Node (nvm), Bun, Python (pyenv), Go, Rust |
| [Phase 6 — Containers](Linux.md#phase-6--containers-docker-engine) | Docker Engine + Compose plugin (native) |
| [Phase 7 — DevOps CLI](Linux.md#phase-7--devops-cli) | kubectl, Terraform |
| [Phase 8 — Recommended extras](Linux.md#phase-8--recommended-extras) | ripgrep, bat, eza, jq, lazygit, … |
| [Phase 9 — Cursor agent skills](Linux.md#phase-9--cursor-agent-skills-optional) | Optional AI workflow skills |
| [Phase 10 — CodeGraph](Linux.md#phase-10--codegraph-optional) | Optional local code knowledge graph (MCP) |
| [Phase 11 — Context7](Linux.md#phase-11--context7-optional) | Optional up-to-date library docs for agents (MCP) |
| [Verification](Linux.md#final-verification-checklist) · [Troubleshooting](Linux.md#troubleshooting) · [Maintenance](Linux.md#maintenance) | Checks, fixes, upkeep |

---

## What you end up with

| Category          | macOS                                    | Ubuntu                                    |
| ----------------- | ---------------------------------------- | ----------------------------------------- |
| Security & access | Bitwarden, Shadowrocket, GitHub          | Bitwarden, Clash Verge / sing-box, GitHub |
| Build base        | Xcode + Command Line Tools               | build-essential + dev headers             |
| Package manager   | Homebrew                                 | apt (Homebrew optional)                   |
| Shell             | zsh, Oh My Zsh, Powerlevel10k, fzf, fd   | same (zsh installed manually)             |
| Languages         | Node (nvm), Bun, Python (pyenv), Go, Rust | same                                     |
| Containers        | Colima + Docker CLI                      | Docker Engine + Compose plugin            |
| DevOps CLI        | git, GitHub CLI, kubectl, Terraform      | same                                      |
| GUI apps          | Cursor, Chrome, Postman, Obsidian, Atomic Chat | Cursor, Chrome, Postman, Obsidian, Ollama + Open WebUI |
| AI terminal       | Terax (`.dmg`)                           | Terax (`.deb` / AppImage)                 |
| AI code intel     | CodeGraph (local MCP knowledge graph)    | same                                      |
| AI docs           | Context7 (up-to-date library docs)       | same                                      |

---

## How to use these guides

1. Pick the guide for your OS.
2. Work through the phases in order — Phase 0 through 7 are the core; 8–11 are optional.
3. Run each **Verify** block before moving on; a failed verify is much cheaper to fix immediately.
4. Finish with the [final verification checklist](Mac.md#final-verification-checklist) ([Linux](Linux.md#final-verification-checklist)).

Every phase is self-contained, so you can also treat these as a reference when setting up a single tool on an existing machine.

---

## License

These guides are free to copy and adapt. Tool licenses belong to their respective vendors.
