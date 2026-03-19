---
layout: default
title: Installation
---

# Installation

## macOS

### Via DMG (recommande)

1. Telecharger le `.dmg` depuis la [page de releases](https://github.com/jiiro974/clientshell-releases/releases/latest)
2. Ouvrir le DMG et glisser ClientShell dans Applications
3. Au premier lancement, macOS peut demander une autorisation (le binaire est signe et notarise)

### Via Homebrew (a venir)

```bash
brew tap jiiro974/clientshell
brew install clientshell
```

### Via binaire direct

```bash
# Telecharger le binaire universel (arm64 + amd64)
curl -L -o clientshell \
  https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-darwin-universal

chmod +x clientshell
sudo mv clientshell /usr/local/bin/
```

---

## Linux

### Debian / Ubuntu (.deb)

```bash
curl -L -o clientshell.deb \
  https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-linux-amd64.deb

sudo dpkg -i clientshell.deb
```

### RHEL / Fedora (.rpm)

```bash
curl -L -o clientshell.rpm \
  https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-linux-amd64.rpm

sudo rpm -i clientshell.rpm
```

### Binaire direct

```bash
curl -L -o clientshell \
  https://github.com/jiiro974/clientshell-releases/releases/latest/download/clientshell-linux-amd64

chmod +x clientshell
sudo mv clientshell /usr/local/bin/
```

---

## Windows

1. Telecharger le `.msi` ou `.exe` depuis les [releases](https://github.com/jiiro974/clientshell-releases/releases/latest)
2. Lancer l'installateur
3. ClientShell sera disponible dans le PATH

---

## Verification de l'installation

```bash
clientshell --version
clientshell health
```

La commande `health` verifie la connectivite au serveur et l'etat des composants.
