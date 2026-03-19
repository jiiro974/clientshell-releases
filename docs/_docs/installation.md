---
layout: default
title: Installation
---

# Installation

## Prerequis

- **Go 1.22+** (pour compiler depuis les sources)
- **Docker** (optionnel, pour les VMs clients)
- **macOS / Linux / Windows**

## Binaires pre-compiles

Telechargez la derniere release :

```bash
# macOS (Apple Silicon)
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-darwin-arm64
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/cs-toolbox-darwin-arm64
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/cs-server-darwin-arm64
chmod +x clientshell-darwin-arm64 cs-toolbox-darwin-arm64 cs-server-darwin-arm64

# Linux (amd64)
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-linux-amd64
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/cs-toolbox-linux-amd64
curl -LO https://github.com/jiiro974/clientshell-releases/releases/latest/download/cs-server-linux-amd64
chmod +x clientshell-linux-amd64 cs-toolbox-linux-amd64 cs-server-linux-amd64

# Windows (amd64) — telecharger les .exe depuis la page Releases
```

## Depuis les sources

```bash
git clone https://github.com/jiiro974/clientshell.git
cd clientshell
make build          # compile les 3 binaires dans bin/
make install        # installe dans /usr/local/bin/
```

### Options d'installation

```bash
make install PREFIX=~/.local       # installe dans ~/.local/bin/
make install PREFIX=/opt/cs        # installe dans /opt/cs/bin/
make uninstall                     # desinstalle
```

## macOS — Gatekeeper

Les binaires non notarises declenchent un avertissement Gatekeeper. Pour autoriser :

```bash
# Option 1 : retirer la quarantaine
make unquarantine

# Option 2 : via Preferences Systeme
# Confidentialite et securite → Autoriser quand meme

# Option 3 : xattr manuellement
xattr -d com.apple.quarantine clientshell-darwin-arm64
```

## Verification

```bash
clientshell version
cs-toolbox version
cs-toolbox help
```

## Autocompletion

```bash
# Bash
cs-toolbox completion bash >> ~/.bashrc

# Zsh
cs-toolbox completion zsh >> ~/.zshrc
```

## Docker

```bash
# Demo avec 6 clients fictifs
make docker-up        # demarre l'infra Docker Compose
./scripts/demo-seed.sh  # provisionne users, incidents, credentials
```

## Mise a jour

```bash
# Depuis les sources
git pull && make build && make install

# Plugins
cs-toolbox upgrade --all
```
