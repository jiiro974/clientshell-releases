---
layout: default
title: Architecture & Securite
---

# Architecture & Securite

## Vue d'ensemble

```
22 packages Go | Zero dependance externe | 100% stdlib

┌─────────────────────── User Layer ───────────────────────┐
│  clientshell CLI    Web Dashboard    Interactive Shell    │
└──────────────────────────┬───────────────────────────────┘
                           │ HTTP/JSON + HMAC-SHA256
┌──────────────────────────┴───────────────────────────────┐
│                    Service Layer                          │
│  Auth RBAC    Audit SHA-256    Secrets    Session Rec     │
│  Toolbox HTTP :9090    cs-server :8080    WebSocket Hub   │
└──────────────────────────┬───────────────────────────────┘
                           │
┌──────────────────────────┴───────────────────────────────┐
│                  Infrastructure Layer                     │
│  Docker VMs    KVM VMs    cs-toolbox 70+    Keeper/Vault  │
└──────────────────────────────────────────────────────────┘
```

## Packages

| Package | Responsabilite |
|---------|---------------|
| `audit` | SHA-256 hash chain immutable, file locking |
| `auth` | RBAC 4 roles, workflow d'approbation |
| `bastion` | Connecteurs Wallix, BeyondTrust |
| `browser` | Enregistrement CDP, DOM snapshots |
| `cli` | Shell interactif, readline, circuit breaker |
| `client` | Configuration clients |
| `crypto` | E2EE (X25519, AES-256-GCM, Ed25519) |
| `demo` | Seed data (8 users, 6 clients, incidents) |
| `distributed` | Dispatcher, pipeline, agent registry |
| `platform` | Abstraction cross-platform (Linux/macOS/Windows) |
| `query` | Moteur SQL sur l'etat systeme |
| `secrets` | Keeper, Vault, rotation |
| `security` | Moteur de regles securite |
| `session` | Enregistrement + replay (asciinema, CDP) |
| `sysmon` | procmon, autoruns, tcpview |
| `toolbox` | 70+ outils, serveur HTTP, plugins, setup |
| `vm` | Providers Docker/KVM/Demo, health |
| `web` | REST API, WebSocket, pagination |

## Securite

### Chiffrement E2E

Tous les composants communiquent via un canal chiffre :

1. **Handshake** : X25519 (Diffie-Hellman sur Curve25519)
2. **Chiffrement** : AES-256-GCM avec nonce unique par message
3. **Signature** : Ed25519 pour l'identite des pairs
4. **Forward secrecy** : cles ephemeres par session

### Authentification

- **HMAC-SHA256** : token signe sur chaque requete toolbox
- **Nonce anti-replay** : 16 bytes crypto/rand, usage unique
- **Rate limiting** : 100 req/min par utilisateur
- **Secret rotation** : current + previous, transition douce

### Audit Trail

- Chaque action produit une entree dans le trail immutable
- Hash chain SHA-256 : chaque entree hash la precedente
- File locking : `flock` (Unix) pour ecriture process-safe
- Verification : `clientshell verify <client>` detecte la falsification

### RBAC

| Role | VM | Clients | Logs | Users | Approbation |
|------|----|---------|------|-------|-------------|
| Admin | CRUD | CRUD | RW | CRUD | Approuver |
| Operator | Start/Stop | Read | Read | - | Demander |
| Supervisor | Start/Stop | Read | Read | - | Approuver |
| Auditor | - | Read | Read/Export | - | - |

## Mode distribue

```
┌─────────────┐
│ Dispatcher   │──── Broadcast ────┬──── Agent ACME
│ (Shell CLI)  │                   ├──── Agent BetaCorp
│              │──── Group(tag) ───├──── Agent Nexus
│              │                   └──── Agent Orion
│              │──── Pipeline ────────── A → B → C
└─────────────┘
```

- **Broadcast** : meme commande sur tous les agents
- **Group** : filtrage par tag (industrie, risque)
- **Pipeline** : chaine de commandes (output → input)
- **Registry** : persistence JSON, heartbeat, status
