---
layout: default
title: Installation
---

# Installation

## Homebrew (recommande pour macOS)

La methode la plus simple — pas de quarantaine Gatekeeper :

```bash
brew tap jiiro974/clientshell
brew install clientshell
```

Cela installe les 3 binaires + autocompletion zsh/bash :
- `clientshell` — CLI interactif
- `cs-toolbox` — 70+ outils SOC
- `cs-server` — Serveur central

Pour installer uniquement cs-toolbox :

```bash
brew install jiiro974/clientshell/cs-toolbox
```

Mise a jour :

```bash
brew upgrade clientshell
```

## Prerequis (installation manuelle)

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

# IMPORTANT: retirer la quarantaine macOS (sinon Gatekeeper bloque)
xattr -d com.apple.quarantine clientshell-darwin-arm64
xattr -d com.apple.quarantine cs-toolbox-darwin-arm64
xattr -d com.apple.quarantine cs-server-darwin-arm64

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

Les binaires telecharges depuis GitHub declenchent un avertissement Gatekeeper car ils ne sont pas notarises par Apple. Trois solutions :

**Solution 1 : Installer via Homebrew (recommande)** — aucune quarantaine :

```bash
brew tap jiiro974/clientshell && brew install clientshell
```

**Solution 2 : Retirer la quarantaine manuellement** :

```bash
# Apres telechargement depuis GitHub
xattr -d com.apple.quarantine cs-toolbox-darwin-arm64
xattr -d com.apple.quarantine clientshell-darwin-arm64
xattr -d com.apple.quarantine cs-server-darwin-arm64

# Ou pour un build local
make unquarantine
```

**Solution 3 : Via Preferences Systeme** :

Apres le premier lancement bloque, aller dans **Reglages Systeme → Confidentialite et securite → Autoriser quand meme**.

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
