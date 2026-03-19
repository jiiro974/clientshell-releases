---
layout: default
title: cs-toolbox — 70+ outils SOC
---

# cs-toolbox — Reference des outils

## Flags globaux

```
--json          Sortie JSON (machine-readable)
--no-color      Desactiver les couleurs
-h, --help      Aide (globale ou par commande)
```

## Reseau

| Commande | Description | Exemple |
|----------|-------------|---------|
| `ping <host>` | Resolution DNS + reachability | `cs-toolbox ping 8.8.8.8` |
| `nslookup <domain>` | DNS A, AAAA, MX, NS, SOA | `cs-toolbox nslookup example.com` |
| `traceroute <host>` | Trace reseau avec latence | `cs-toolbox traceroute google.com` |
| `netstat` | Interfaces reseau | `cs-toolbox netstat` |
| `port-check <host:port>` | Test connectivite TCP/UDP | `cs-toolbox port-check google.com:443` |
| `portscan <host> [spec]` | Scanner de ports concurrent | `cs-toolbox portscan host top100` |
| `subnet <cidr>` | Calculateur CIDR | `cs-toolbox subnet 10.0.0.0/24` |
| `nc <proto> <host:port>` | Netcat + banner grabbing | `cs-toolbox nc tcp host:80` |
| `myip` | IP publique + reverse DNS | `cs-toolbox myip` |
| `whois <domain\|ip>` | WHOIS avec suivi d'expiration | `cs-toolbox whois example.com` |
| `dns-enum <domain> [type]` | Enumeration DNS | `cs-toolbox dns-enum example.com MX` |
| `cert-check <host[:port]>` | Inspection certificat TLS | `cs-toolbox cert-check google.com` |
| `http-headers <url>` | Analyse headers securite | `cs-toolbox http-headers https://example.com` |
| `ssl-audit <host[:port]>` | Audit TLS/SSL + notation | `cs-toolbox ssl-audit google.com` |
| `mx-toolbox <domain>` | Infra email (SPF/DMARC/DKIM) | `cs-toolbox mx-toolbox example.com` |

### netexplorer — Explorateur reseau (ntop-like)

```bash
cs-toolbox netexplorer                    # rapport complet
cs-toolbox netexplorer --proto tcp        # filtrer TCP
cs-toolbox netexplorer --state ESTABLISHED
cs-toolbox netexplorer --top 10           # top 10 talkers
cs-toolbox netexplorer --alerts           # alertes uniquement
cs-toolbox netexplorer --reputation       # reputation IP (0-100)
cs-toolbox netexplorer --watch            # auto-refresh 5s
cs-toolbox netexplorer --watch --interval 2
```

| Flag | Description |
|------|-------------|
| `--proto <tcp\|udp>` | Filtrer par protocole |
| `--state <STATE>` | ESTABLISHED, LISTEN, TIME_WAIT, CLOSE_WAIT, SYN_SENT |
| `--pid <PID>` | Filtrer par PID |
| `--top <N>` | Top N remote hosts (defaut: 10) |
| `--alerts` | Alertes uniquement (ports suspects, IRC, TOR) |
| `--reputation` | Score IP 0-100 (trusted/neutral/suspicious/malicious) |
| `--watch` | Mode auto-refresh |
| `--interval <s>` | Intervalle de refresh (defaut: 5s) |

## Systeme

| Commande | Description |
|----------|-------------|
| `ps` | Processus avec CPU/memoire |
| `df` | Espace disque |
| `meminfo` | Statistiques memoire |
| `sysinfo` | OS, architecture, CPU, uptime |
| `health <type> [args]` | Verification sante multi-service |

### health — Types disponibles

```bash
cs-toolbox health http https://api.example.com
cs-toolbox health tcp host:3306
cs-toolbox health udp host:53
cs-toolbox health ssl google.com:443
cs-toolbox health smtp mail.example.com
cs-toolbox health ws wss://api.example.com/ws
cs-toolbox health vpn gateway.example.com
cs-toolbox health disk
cs-toolbox health memory
cs-toolbox health system
```

### files — Explorateur fichiers ouverts (lsof-like)

```bash
cs-toolbox files                          # rapport complet
cs-toolbox files --user root              # filtrer par user
cs-toolbox files --process nginx          # filtrer par process
cs-toolbox files --type SOCK              # filtrer par type
cs-toolbox files --alerts                 # fichiers suspects uniquement
cs-toolbox files --top 10                 # top 10 par nombre de FDs
```

## Securite

| Commande | Description |
|----------|-------------|
| `hash <file>` | MD5 + SHA-256 |
| `check-hash <file> <hash>` | Verification integrite |
| `scan <target>` | IOC scan (patterns regex) |
| `ioc-scan <path>` | Scan IOC approfondi avec severite |
| `ip-lookup <ip>` | Reputation IP |
| `domain-lookup <domain>` | Analyse menace domaine |
| `secret-scan [mode]` | Detection credentials (files/env/all) |
| `fim <action> [path]` | Monitoring integrite (baseline/check) |
| `log-analyze <file\|system>` | Analyse logs securite |
| `base64 <mode> <input>` | Encode/decode (encode/decode/auto/hex/file) |
| `timeline <file>` | Timeline forensique |
| `yara-scan <path>` | Scanner malware YARA-like |
| `screenshot <url>` | Capture page web via HTTP |

### screenshot — Capture web

```bash
cs-toolbox screenshot https://example.com
cs-toolbox screenshot https://example.com -o page.html --format html
cs-toolbox screenshot https://example.com --format headers
cs-toolbox screenshot https://10.0.0.1:8443 --insecure
```

| Flag | Description |
|------|-------------|
| `-o, --output <file>` | Fichier de sortie (defaut: capture.html) |
| `--format <html\|headers\|full>` | Format (defaut: full) |
| `--timeout <s>` | Timeout (defaut: 15s) |
| `--user-agent <ua>` | User-Agent personnalise |
| `--insecure` | Ignorer verification TLS |
| `--no-follow` | Ne pas suivre les redirections |

## Query (SQL)

```bash
cs-toolbox query "SELECT * FROM processes WHERE user = 'root'"
cs-toolbox query "SELECT pid, name FROM processes ORDER BY pid LIMIT 10"
cs-toolbox query ".tables"
cs-toolbox query ".schema processes"
```

### Tables disponibles

| Table | Colonnes |
|-------|----------|
| `processes` | pid, name, user, cpu, mem, path |
| `listening_ports` | proto, local_addr, local_port, pid |
| `connections` | proto, local_addr, remote_addr, state, pid |
| `users` | name, uid, gid, home, shell |
| `mounts` | device, mount, type, options |
| `interfaces` | name, ip, mac, flags |
| `env` | key, value |

## Monitoring

```bash
cs-toolbox procmon                    # processus
cs-toolbox procmon --tree             # arbre parent/enfant
cs-toolbox procmon --sort cpu --top 10
cs-toolbox autoruns                   # mecanismes de persistance
cs-toolbox autoruns --new             # ajoutes < 7 jours
cs-toolbox tcpview                    # connexions TCP temps reel
cs-toolbox tcpview --diff             # new/closed depuis dernier snapshot
```

## Plugins

```bash
cs-toolbox plugin list                # plugins installes
cs-toolbox plugin install <path>      # installer un plugin
cs-toolbox plugin remove <name>       # supprimer
cs-toolbox plugin verify              # verifier SHA-256
cs-toolbox setup --bundle soc         # installer un bundle
cs-toolbox setup --list               # lister tout
cs-toolbox upgrade                    # mettre a jour tout
cs-toolbox upgrade <name>             # mettre a jour un plugin
```

### Bundles disponibles

| Bundle | Contenu |
|--------|---------|
| `minimal` | ping, nslookup, portscan, dns-enum, subnet |
| `network` | minimal + traceroute, whois, cert-check, ssl-audit, http-headers, mx-toolbox, netexplorer, myip, netcat |
| `soc` | ping, nslookup, portscan + ioc-scan, yara-scan, secret-scan, log-analyze, fim, timeline, hash, base64, netexplorer, files |
| `full` | tous les outils |
