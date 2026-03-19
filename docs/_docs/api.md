---
layout: default
title: API Reference
---

# API Reference

## cs-server — REST API

**Base URL** : `http://localhost:8080/api`

### Endpoints

| Methode | Path | Description | Auth |
|---------|------|-------------|------|
| GET | `/api/health` | Etat du serveur | Non |
| GET | `/api/status` | Statut detaille | Non |
| POST | `/api/auth/login` | Authentification | Non |
| GET | `/api/users` | Lister les utilisateurs | Bearer |
| POST | `/api/users` | Creer un utilisateur | Bearer |
| GET | `/api/clients` | Lister les clients | Bearer |
| POST | `/api/clients` | Enregistrer un client | Bearer |
| GET | `/api/recordings` | Lister les enregistrements | Bearer |
| GET | `/api/recordings/{id}` | Detail d'un enregistrement | Bearer |
| GET | `/api/recordings/{id}/events` | Evenements d'une session | Bearer |

### WebSocket

**URL** : `ws://localhost:8080/ws`

Evenements temps reel :
- `session_recording_started`
- `session_event` (commande, sortie, navigation)
- `session_recording_ended`
- `chat_message`
- `presence` (join, leave)

## cs-toolbox — HTTP API

**Base URL** : `http://localhost:9090`

### Endpoints

| Methode | Path | Description | Auth |
|---------|------|-------------|------|
| GET | `/health` | Sante du service | Non |
| GET | `/tools` | Lister les outils | Bearer HMAC |
| POST | `/exec` | Executer un outil | Bearer HMAC |

### POST /exec

```json
// Requete
{
  "command": "portscan",
  "args": ["scanme.nmap.org", "top100"]
}

// Reponse
{
  "output": { ... },
  "exit_code": 0,
  "error": ""
}
```

### GET /health

```json
{
  "status": "ok",
  "version": "1.0.0",
  "uptime_seconds": 3600,
  "tools_count": 43,
  "requests_total": 1234,
  "requests_failed": 5
}
```

### Authentification HMAC

Chaque requete doit porter un token HMAC-SHA256 :

```
Authorization: Bearer <token>
X-Request-ID: <uuid>
```

Le token contient : user, role, client_id, nonce, expiry, issued_at.
