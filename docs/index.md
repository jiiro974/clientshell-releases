---
layout: default
title: ClientShell
---

<div style="text-align: center; padding: 2rem 0;">
  <h1 style="font-size: 2.5rem; margin-bottom: 0.5rem;">ClientShell</h1>
  <p style="font-size: 1.2rem; color: #666; max-width: 700px; margin: 0 auto 1rem;">
    Plateforme securisee de gestion de postes de travail pour equipes MSP/SOC.<br>
    <strong>Go stdlib only. Zero dependance externe. 70+ outils.</strong>
  </p>

  <p style="background: #f0f0f0; padding: 12px 20px; border-radius: 8px; font-family: monospace; font-size: 1rem; display: inline-block; margin-bottom: 1rem;">
    brew tap jiiro974/clientshell && brew install clientshell
  </p>

  <div>
  <a href="https://github.com/jiiro974/clientshell-releases/releases/latest"
     style="display: inline-block; padding: 12px 28px; background: #2ea44f; color: white; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 1.1rem; margin: 0.5rem;">
    Telecharger
  </a>
  <a href="/clientshell-releases/docs/installation/"
     style="display: inline-block; padding: 12px 28px; background: #0969da; color: white; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 1.1rem; margin: 0.5rem;">
    Installation
  </a>
  <a href="/clientshell-releases/docs/quickstart/"
     style="display: inline-block; padding: 12px 28px; background: #6f42c1; color: white; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 1.1rem; margin: 0.5rem;">
    Guide demarrage
  </a>
  <a href="/clientshell-releases/docs/demos/"
     style="display: inline-block; padding: 12px 28px; background: #e36209; color: white; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 1.1rem; margin: 0.5rem;">
    Voir les demos
  </a>
  </div>
</div>

---

## Vue d'ensemble

ClientShell est une plateforme Go production-ready pour gerer des postes de travail securises. Elle fournit :

| Composant | Description |
|-----------|-------------|
| **clientshell** | CLI interactif avec 20+ commandes builtin |
| **cs-toolbox** | 70+ outils SOC/securite + serveur HTTP API |
| **cs-server** | Serveur central REST + WebSocket temps reel |

## Architecture

```
22 packages Go | 3 binaires | 3 plateformes (Linux, macOS, Windows)
├── Microservice : Shell + Toolbox via HTTP/JSON + HMAC auth
├── E2EE : X25519 + AES-256-GCM + Ed25519
├── Audit : SHA-256 hash chain immutable
├── Distribue : broadcast, group, pipeline multi-agents
├── Query : SQL sur l'etat systeme (osquery-like)
└── Plugins : systeme modulaire avec setup wizard
```

## Fonctionnalites principales

- **VM management** : Docker + KVM, 3 modes (shell, desktop, browser)
- **70+ outils SOC** : netexplorer, files, portscan, whois, yara-scan, secret-scan...
- **Shell interactif** : readline, autocompletion, historique persistant
- **Chiffrement E2E** : X25519 + AES-256-GCM entre tous les composants
- **Audit immutable** : SHA-256 hash chain avec anti-falsification
- **Multi-agent** : broadcast, group, pipeline sur N agents distants
- **SQL engine** : `SELECT * FROM processes WHERE user = 'root'`
- **Cross-platform** : Linux, macOS (signe), Windows

## Plateformes

| Plateforme | Architecture | Signe |
|------------|-------------|-------|
| macOS | arm64, amd64, universal | Code sign + entitlements |
| Linux | amd64, arm64 | - |
| Windows | amd64, arm64 | - |

## Liens rapides

- [Installation](/clientshell-releases/docs/installation/)
- [Guide demarrage rapide](/clientshell-releases/docs/quickstart/)
- [Reference des commandes](/clientshell-releases/docs/commands/)
- [cs-toolbox (70+ outils)](/clientshell-releases/docs/toolbox/)
- [Architecture & securite](/clientshell-releases/docs/architecture/)
- [API Reference](/clientshell-releases/docs/api/)
- [Releases](https://github.com/jiiro974/clientshell-releases/releases)
