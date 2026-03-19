---
layout: default
title: Reference des commandes
---

# Reference des commandes clientshell

## Commandes principales

| Commande | Description |
|----------|-------------|
| `connect <client>` | Se connecter a un client (shell interactif par defaut) |
| `disconnect <client>` | Deconnecter et arreter la VM |
| `status [client]` | Statut des sessions actives |
| `status --all` | Toutes les sessions avec details |
| `--all` | Dashboard complet (identite, sessions, audit) |
| `logs <client>` | Consulter les logs d'audit |
| `verify <client>` | Verifier l'integrite du trail SHA-256 |
| `exec <vm-id> <cmd>` | Executer une commande dans une VM |
| `invite <user>` | Inviter un collegue dans la session |
| `join <id>` | Rejoindre une session partagee |
| `observe <id>` | Observer une session en lecture seule |
| `inspect [session]` | Inspecter les sessions (superviseur) |
| `kill <session>` | Terminer une session d'urgence |
| `version` | Version de clientshell |
| `help` | Aide |

## Options de connexion

```bash
clientshell connect acme-corp                     # shell interactif (defaut)
clientshell connect acme-corp --mode desktop       # bureau VNC/noVNC
clientshell connect acme-corp --mode browser        # navigateur avec enregistrement CDP
clientshell connect acme-corp --basic              # docker exec brut (legacy)
clientshell connect acme-corp --rebuild            # detruire et recreer le conteneur
clientshell connect acme-corp --image custom:latest # image Docker personnalisee
```

## Commandes du shell interactif

### Navigation
| Commande | Alias | Description |
|----------|-------|-------------|
| `:help` | `:h`, `:?` | Aide complete |
| `:quit` | `:q`, `:exit` | Quitter la session |
| `:clear` | `:cls` | Effacer l'ecran |
| `:history` | `:hist` | Historique des commandes |

### Information
| Commande | Description |
|----------|-------------|
| `:status` | Etat rapide de la session |
| `:whoami` | Identite et role RBAC |
| `:info` | Details de la session |
| `:who` | Participants connectes |
| `:env` | Variables d'environnement |
| `:version` | Version + plateforme |
| `:uptime` | Duree de session |
| `:clients` | Sessions clients actives |

### Outils systeme
| Commande | Description |
|----------|-------------|
| `:df` | Espace disque |
| `:mem` | Memoire |
| `:ps` | Processus |
| `:sysinfo` | Informations systeme |
| `:health <type>` | Verification sante |
| `:netmon` | Explorateur reseau |
| `:files` | Fichiers ouverts |
| `:screenshot <url>` | Capture page web |
| `:tools` | Liste des 70+ outils |
| `:toolbox <cmd>` | Commande cs-toolbox |

### Collaboration
| Commande | Description |
|----------|-------------|
| `:chat <msg>` | Message aux participants |
| `:invite <user>` | Inviter un collegue |
| `:invitations` | Invitations en attente |

### Fichiers
| Commande | Description |
|----------|-------------|
| `:upload <file>` | Envoyer un fichier vers la VM |
| `:download <file>` | Telecharger depuis la VM |

### Audit
| Commande | Description |
|----------|-------------|
| `:logs` | Logs d'audit du client |
| `:verify` | Verification integrite SHA-256 |

### Multi-agents
| Commande | Description |
|----------|-------------|
| `:agents` | Lister les agents distants |
| `:broadcast <cmd>` | Executer sur tous les agents |
| `:agent <name> <cmd>` | Executer sur un agent specifique |

### Raccourcis clavier
| Touche | Action |
|--------|--------|
| ↑ / ↓ | Historique |
| Ctrl-A | Debut de ligne |
| Ctrl-E | Fin de ligne |
| Ctrl-W | Supprimer mot |
| Ctrl-U | Supprimer jusqu'au debut |
| Ctrl-K | Supprimer jusqu'a la fin |
| Ctrl-L | Effacer ecran |
| Tab | Autocompletion |
