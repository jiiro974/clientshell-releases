---
layout: default
title: Configuration
---

# Configuration

## Fichier de configuration

ClientShell utilise un fichier YAML situe a `~/.clientshell/config.yml`.

```yaml
# Serveur ClientShell
server:
  url: "https://clientshell.example.com"
  port: 8443

# Authentification
auth:
  method: "bearer"    # bearer, cookie, ou oidc
  # token: "xxx"      # Optionnel, sinon login interactif

# Sessions
session:
  provider: "docker"  # docker ou kvm
  timeout: 3600       # Duree max en secondes
  recording: true     # Enregistrement automatique

# Securite
security:
  auto_scan: true           # Scan automatique des containers
  scan_on_connect: true     # Scan a chaque connexion
  require_approval: false   # Workflow d'approbation

# Logs
logging:
  level: "info"       # debug, info, warn, error
  file: "~/.clientshell/logs/clientshell.log"
```

## Variables d'environnement

Les variables d'environnement ont priorite sur le fichier de config.

| Variable | Description | Defaut |
|:---|:---|:---|
| `CLIENTSHELL_SERVER_URL` | URL du serveur | — |
| `CLIENTSHELL_AUTH_TOKEN` | Token d'authentification | — |
| `CLIENTSHELL_LOG_LEVEL` | Niveau de log | `info` |
| `CLIENTSHELL_PROVIDER` | Provider de sessions | `docker` |
| `CLIENTSHELL_CONFIG` | Chemin vers le fichier de config | `~/.clientshell/config.yml` |

## Premiere configuration

```bash
# Configuration interactive
clientshell init

# Ou configuration manuelle
clientshell config set server.url https://clientshell.example.com
clientshell config set auth.method bearer
```

## Profils

Gerez plusieurs environnements avec des profils :

```bash
# Creer un profil
clientshell profile create staging --server https://staging.example.com

# Basculer entre profils
clientshell profile use staging
clientshell profile use production

# Lister les profils
clientshell profile list
```
