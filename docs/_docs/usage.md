---
layout: default
title: Utilisation
---

# Utilisation

## Connexion au serveur

```bash
# Login interactif
clientshell login

# Login avec token
clientshell login --token <your-token>

# Verifier la connexion
clientshell whoami
```

## Gestion des sessions

```bash
# Lister les sessions disponibles
clientshell sessions list

# Demarrer une nouvelle session
clientshell session start --provider docker

# Se connecter a une session existante
clientshell session connect <session-id>

# Terminer une session
clientshell session stop <session-id>
```

## Enregistrements

```bash
# Lister les enregistrements
clientshell recordings list

# Lire un enregistrement
clientshell recordings play <recording-id>

# Exporter un enregistrement
clientshell recordings export <recording-id> --format mp4 --output ./recording.mp4
```

## Scans de securite

```bash
# Lancer un scan
clientshell scan run

# Voir les resultats
clientshell scan results

# Scan d'un container specifique
clientshell scan run --target <container-id>
```

## Administration (roles admin/manager)

```bash
# Gestion des utilisateurs
clientshell users list
clientshell users create --email user@example.com --role operator

# Gestion des approbations
clientshell approvals pending
clientshell approvals approve <request-id>
clientshell approvals deny <request-id> --reason "Non conforme"

# Audit
clientshell audit logs --since 24h
clientshell audit export --format csv --output audit.csv
```

## Commandes utiles

```bash
# Etat du systeme
clientshell health
clientshell status

# Version
clientshell --version

# Aide
clientshell --help
clientshell <command> --help
```
