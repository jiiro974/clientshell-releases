---
layout: default
title: Demarrage rapide
---

# Demarrage rapide

## 1. Lancer le shell interactif

```bash
./clientshell
```

Sans arguments, ClientShell demarre en mode interactif avec un DemoProvider integre. Tapez `:help` pour voir les commandes disponibles.

## 2. Se connecter a un client

```bash
./clientshell connect acme-corp
```

Cela cree un conteneur Docker, provisionne zsh + cs-toolbox, et vous donne un shell interactif.

## 3. Commandes du shell

```bash
:help                    # aide complete
:status                  # etat de la session
:whoami                  # identite + role RBAC
:tools                   # liste des 70+ outils
:netmon                  # explorateur reseau (ntop-like)
:files                   # fichiers ouverts (lsof-like)
:quit                    # quitter
```

## 4. Outils cs-toolbox

```bash
cs-toolbox portscan scanme.nmap.org top100
cs-toolbox whois example.com
cs-toolbox netexplorer --watch --reputation
cs-toolbox files --alerts
cs-toolbox health ssl google.com:443
cs-toolbox screenshot https://example.com -o page.html
cs-toolbox log-analyze system
cs-toolbox query "SELECT pid, name FROM processes LIMIT 10"
cs-toolbox secret-scan all
```

## 5. Dashboard

```bash
./clientshell --all     # dashboard complet
./clientshell status --all  # sessions detaillees
```

## 6. Mode multi-agents

```bash
# Dans le shell interactif :
:agents add acme http://10.200.0.10:9090
:agents add betacorp http://10.200.0.11:9090
:broadcast portscan top100
:agent acme netexplorer --alerts
```

## 7. Demo complete

```bash
make docker-up           # 6 clients Docker
./scripts/demo-seed.sh   # 8 users, 12 incidents
CS_USER=jrousseau CS_ROLE=admin ./clientshell
```

## Variables d'environnement

| Variable | Description | Default |
|----------|-------------|---------|
| `CS_USER` | Utilisateur courant | `$USER` |
| `CS_ROLE` | Role RBAC | `operator` |
| `CS_AUDIT_DIR` | Repertoire audit logs | `./audit-logs` |
| `CS_VM_PROVIDER` | Provider VM | `docker` |
| `CS_TOOLBOX_URL` | URL du service toolbox | auto-detect |
| `CS_SERVER_URL` | URL du serveur central | - |
| `CS_AUTH_SECRET` | Secret HMAC pour auth | - |
| `CS_LOG_LEVEL` | Niveau de log | `info` |
