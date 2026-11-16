# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Overview

Personal dotfiles managed with [Chezmoi](https://chezmoi.io) (file management/templating) and [Ansible](https://github.com/ansible/ansible) (system provisioning). The philosophy is **minimal customization** to stay productive on shared environments.

> **Managed-file policy.** Files this repo deploys into `$HOME` are generated output — editing the live copies causes drift. Before creating or editing any file under `$HOME`, or when you see a "Managed by chezmoi" disclaimer at the top of a file, read and follow [`docs/managed-dotfiles-policy.md`](docs/managed-dotfiles-policy.md): edit the source (not the live copy), reconcile drift with `chezmoi re-add` (not `apply --force`), and add the standard disclaimer to comment-capable managed files.

## Chezmoi file naming conventions

Chezmoi uses filename prefixes/suffixes to determine how files are handled:

- `dot_*` → becomes a dotfile (e.g., `dot_bashrc` → `~/.bashrc`)
- `run_once_*` → script run once on first apply
- `run_onchange_*` → script re-run whenever its content changes
- `*.tmpl` → processed as a Go template before applying
- `dot_bootstrap/` → becomes `~/.bootstrap/` (Ansible playbook location)

## Architecture

### Bootstrap flow

1. Chezmoi is initialized via `sh -c "$(curl -fsLS get.chezmoi.io)" -- init --apply $GITHUB_USERNAME`
2. `run_once_install_ansible.sh` installs Ansible (supports macOS/Debian/Fedora; on WSL the Debian path is used). It only installs Ansible — it does not run any playbook
3. `run_onchange_provision.sh.tmpl` runs the platform-specific provisioning, and re-runs whenever it or the platform's playbook changes (sha256 of the playbook in the script header is the change trigger). It runs with `set -euo pipefail`, so a failed playbook fails `chezmoi apply` and is retried on the next apply rather than being recorded as done
4. Provisioning files in `dot_bootstrap/`:
   - `provision-workstation-macos.yml` — macOS (Ansible)
   - `provision-workstation-linux.yml` — native Linux (Ansible)
   - `provision-workstation-wsl.yml` — the Linux side of WSL (Ansible: apt packages, gh, Claude Code)
   - `provision-workstation-windows.ps1` — the Windows side of WSL (PowerShell + winget: VS Code, 7-Zip, JetBrains Mono Nerd Font), run via `powershell.exe` after the WSL playbook

### Template data

`.chezmoi.toml.tmpl` prompts for `machine_profile` (`workstation` or `server`), `name`, `email`, and `githubUser` on first run. These values are available in templates as `{{ .chezmoi.config.data.name }}` etc.

`email` is not just config — `dot_gitconfig.tmpl` makes it the git commit identity, so it is published in every commit the machine makes and cannot be recalled once pushed. Answer it with a role address on a domain you control rather than a personal inbox: portable off any one host, and safe to publish. A personal address here propagates to every repository on the machine silently, and because the live `~/.gitconfig` is generated, a per-repo `git config user.email` override is reverted by the next `chezmoi apply` — fix the template, not the output. There is deliberately **no default**: this file is committed, so a hardcoded address would become the identity of anyone who forks these dotfiles.

### Shell config

- `dot_shellrc` → shared POSIX-compatible config sourced by both `.bashrc` and `.zshrc`; sets `EDITOR`, `HISTSIZE`, PATH, and optional tool integrations (pyenv, nvm)
- `dot_bash_profile` → sources `~/.bashrc` for login shells
- `dot_bashrc` → sources `~/.shellrc`, adds bash-specific history settings and nvm bash completion
- `dot_zshrc` → sources `~/.shellrc`
- `dot_inputrc` → readline config (history search with arrows, case-insensitive completion)

When adding shell config that should apply to both bash and zsh, put it in `dot_shellrc` and keep it POSIX-compatible (no bash/zsh-specific syntax).

## Key commands

```bash
# Init (clones repo into ~/.local/share/chezmoi/ and applies)
chezmoi init --apply $GITHUB_USERNAME

# Init for non-standard local path (then add sourceDir to ~/.config/chezmoi/chezmoi.toml)
chezmoi init --apply --source /local/path/to/dotfiles

# Apply dotfiles from source dir
chezmoi apply

# See what changes would be made
chezmoi diff

# Edit a managed file (opens in $EDITOR, applies on save)
chezmoi edit ~/.bashrc

# Re-run a run_once script after changes
chezmoi state delete-bucket --bucket=scriptOnce && chezmoi apply

# Run Ansible playbook manually
ansible-playbook ~/.bootstrap/provision-workstation-macos.yml --ask-become-pass  # macOS
ansible-playbook ~/.bootstrap/provision-workstation-linux.yml --ask-become-pass  # Linux
ansible-playbook ~/.bootstrap/provision-workstation-wsl.yml --ask-become-pass    # WSL (Linux side)
# On Ubuntu 25.10+ (sudo-rs), prefix with ANSIBLE_BECOME_EXE=sudo.ws — see Notes
```

## Notes

- `.chezmoiignore` excludes `CLAUDE.md`, `README.md`, `docs/`, and `.gitignore` from being deployed to the home directory
- **sudo-rs workaround — do not remove.** Ubuntu 25.10+ ships `sudo-rs` as the default `sudo`. Ansible's become plugin passes a custom `-p` prompt and waits for that exact text, but sudo-rs wraps it as `[sudo: <prompt>] Password:`, so every `become: true` task fails with "Timed out waiting for become success or become password prompt". `run_onchange_provision.sh.tmpl` detects sudo-rs and sets `ANSIBLE_BECOME_EXE=sudo.ws` (classic sudo, still shipped by Ubuntu). It is a no-op on macOS, Fedora and older Ubuntu
- Editing managed files: follow [`docs/managed-dotfiles-policy.md`](docs/managed-dotfiles-policy.md) — edit the chezmoi source, never the live `$HOME` copy
- Changing Claude Code permission rules: follow [`docs/claude-permission-gating.md`](docs/claude-permission-gating.md). The permission gate itself now lives in [luma-foreman](https://github.com/LumaStack/luma-foreman), not here — this repo owns only the native rules and the hook wiring in `dot_claude/settings.json`. Per-project changes are `luma-foreman policy allow <key>` and touch no file in this repository
- The `hands-free` command (Superwhisper + macOS Voice Control for voice-driven sessions), why installing its shortcuts needs one manual click, and the non-obvious Shortcuts serialization it depends on: see [`docs/hands-free.md`](docs/hands-free.md). Run `hands-free init` once per machine — it is deliberately not an Ansible task, so provisioning stays non-interactive
- Kitty tab status icons for Claude Code (▸ working / ⏸ waiting), plus the known "wrong icon" bug and its live diagnostic instrumentation: see [`docs/kitty-claude-tab-icons.md`](docs/kitty-claude-tab-icons.md)