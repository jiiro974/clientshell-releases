---
layout: default
title: ClientShell
---

<div style="text-align: center; padding: 2rem 0;">
  <h1 style="font-size: 2.5rem; margin-bottom: 0.5rem;">ClientShell</h1>
  <p style="font-size: 1.2rem; color: #666; max-width: 600px; margin: 0 auto 2rem;">
    Plateforme MSP/SOC de gestion securisee de postes de travail.
    Connexion distante, enregistrement de sessions, scans de securite.
  </p>

  {% if site.data.latest_release.version != "v0.0.0" %}
  <a href="https://github.com/jiiro974/clientshell-releases/releases/latest"
     style="display: inline-block; padding: 12px 28px; background: #2ea44f; color: white; text-decoration: none; border-radius: 6px; font-weight: bold; font-size: 1.1rem;">
    Telecharger {{ site.data.latest_release.version }}
  </a>
  <p style="margin-top: 0.5rem; color: #888; font-size: 0.9rem;">
    Publie le {{ site.data.latest_release.date }}
  </p>
  {% else %}
  <p style="color: #888;">Premiere release bientot disponible</p>
  {% endif %}
</div>

---

## Fonctionnalites

| Fonctionnalite | Description |
|:---|:---|
| Connexion securisee | Sessions VM via Docker ou KVM avec chiffrement de bout en bout |
| Enregistrement | Capture video des sessions pour audit et conformite |
| Scans de securite | Analyse automatisee des containers et des endpoints |
| RBAC & Approbations | Controle d'acces granulaire avec workflow d'approbation |
| Audit complet | Journal detaille de toutes les actions pour compliance |

## Plateformes supportees

| Plateforme | Architecture | Format |
|:---|:---|:---|
| macOS | Universal (arm64 + amd64) | DMG, binaire |
| Linux | amd64, arm64 | binaire, .deb, .rpm |
| Windows | amd64 | .exe, .msi |

## Telechargements

{% if site.data.latest_release.assets.size > 0 %}

**{{ site.data.latest_release.version }}** — {{ site.data.latest_release.date }}

| Fichier | Taille | Lien |
|:---|:---|:---|
{% for asset in site.data.latest_release.assets %}| {{ asset.name }} | {{ asset.size }} | [Telecharger]({{ asset.url }}) |
{% endfor %}

{% else %}
*Les binaires seront disponibles apres la premiere release.*
{% endif %}

[Voir toutes les releases](https://github.com/jiiro974/clientshell-releases/releases){: .btn }

---

## Documentation

- [Installation]({{ site.baseurl }}/docs/installation/) — Guide d'installation par plateforme
- [Configuration]({{ site.baseurl }}/docs/configuration/) — Parametres et fichiers de config
- [Utilisation]({{ site.baseurl }}/docs/usage/) — Guide d'utilisation quotidien

---

## Demarrage rapide

```bash
# macOS — installer depuis le DMG
open ClientShell-v*.dmg

# Linux — binaire direct
chmod +x clientshell-linux-amd64
sudo mv clientshell-linux-amd64 /usr/local/bin/clientshell

# Verifier l'installation
clientshell --version
clientshell health
```

## Support

- [Issues](https://github.com/jiiro974/clientshell-releases/issues) — Reporter un bug ou demander une fonctionnalite
- [Discussions](https://github.com/jiiro974/clientshell-releases/discussions) — Questions et echanges
