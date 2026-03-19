---
layout: default
title: Demos
---

# Demos — cs-toolbox en action

Chaque demo montre une commande reelle executee avec sa sortie. Cliquez sur le player pour demarrer.

<link rel="stylesheet" type="text/css" href="https://unpkg.com/asciinema-player@3.7.1/dist/bundle/asciinema-player.css" />
<script src="https://unpkg.com/asciinema-player@3.7.1/dist/bundle/asciinema-player.min.js"></script>

<style>
.demo-section { margin: 2rem 0; }
.demo-section h3 { margin-bottom: 0.5rem; color: #0969da; }
.demo-section p { color: #666; font-size: 0.9rem; margin-bottom: 0.5rem; }
.demo-grid { display: grid; grid-template-columns: 1fr; gap: 2rem; }
@media (min-width: 900px) { .demo-grid { grid-template-columns: 1fr 1fr; } }
</style>

## Reseau

<div class="demo-grid">

<div class="demo-section">
<h3>ping — Metriques reseau</h3>
<p>RTT min/avg/max, jitter, packet loss, DNS resolve time</p>
<div id="demo-ping"></div>
</div>

<div class="demo-section">
<h3>nslookup — Resolution DNS</h3>
<p>A, AAAA records avec temps de resolution</p>
<div id="demo-nslookup"></div>
</div>

<div class="demo-section">
<h3>traceroute — Trace reseau</h3>
<p>Hops avec resolution DNS parallele</p>
<div id="demo-traceroute"></div>
</div>

<div class="demo-section">
<h3>port-check — Test de port</h3>
<p>Latence, protocole, banner grab</p>
<div id="demo-port-check"></div>
</div>

<div class="demo-section">
<h3>whois — WHOIS Lookup</h3>
<p>Registrar, expiration, status EPP</p>
<div id="demo-whois"></div>
</div>

<div class="demo-section">
<h3>subnet — Calculateur CIDR</h3>
<p>Plage d'hotes, broadcast, masque</p>
<div id="demo-subnet"></div>
</div>

<div class="demo-section">
<h3>cert-check — Certificat TLS</h3>
<p>Validation, expiration, chaine</p>
<div id="demo-cert-check"></div>
</div>

<div class="demo-section">
<h3>http-headers — Headers securite</h3>
<p>Analyse et scoring des headers HTTP</p>
<div id="demo-http-headers"></div>
</div>

<div class="demo-section">
<h3>ssl-audit — Audit TLS/SSL</h3>
<p>Configuration, grade, vulnerabilites</p>
<div id="demo-ssl-audit"></div>
</div>

<div class="demo-section">
<h3>dns-enum — Enumeration DNS</h3>
<p>Tous les types de records</p>
<div id="demo-dns-enum"></div>
</div>

<div class="demo-section">
<h3>myip — IP publique</h3>
<p>IP publique + reverse DNS</p>
<div id="demo-myip"></div>
</div>

</div>

## Systeme

<div class="demo-grid">

<div class="demo-section">
<h3>ps — Processus</h3>
<p>Liste avec CPU et memoire</p>
<div id="demo-ps"></div>
</div>

<div class="demo-section">
<h3>df — Espace disque</h3>
<p>Utilisation par point de montage</p>
<div id="demo-df"></div>
</div>

<div class="demo-section">
<h3>meminfo — Memoire</h3>
<p>Statistiques RAM et swap</p>
<div id="demo-meminfo"></div>
</div>

<div class="demo-section">
<h3>sysinfo — Systeme</h3>
<p>OS, architecture, CPU, uptime</p>
<div id="demo-sysinfo"></div>
</div>

</div>

## Securite

<div class="demo-grid">

<div class="demo-section">
<h3>secret-scan — Detection de secrets</h3>
<p>Scan des variables d'environnement</p>
<div id="demo-secret-scan"></div>
</div>

<div class="demo-section">
<h3>screenshot — Capture web</h3>
<p>Page capturee via HTTP (sans browser)</p>
<div id="demo-screenshot"></div>
</div>

<div class="demo-section">
<h3>base64 — Encodage</h3>
<p>Encode/decode Base64</p>
<div id="demo-base64"></div>
</div>

</div>

## Query (SQL)

<div class="demo-grid">

<div class="demo-section">
<h3>query .tables — Tables disponibles</h3>
<p>Liste des tables virtuelles</p>
<div id="demo-query-tables"></div>
</div>

<div class="demo-section">
<h3>query — Variables d'environnement</h3>
<p>SELECT * FROM env LIMIT 5</p>
<div id="demo-query-env"></div>
</div>

<div class="demo-section">
<h3>query — Interfaces reseau</h3>
<p>SELECT * FROM interfaces</p>
<div id="demo-query-interfaces"></div>
</div>

</div>

## Autre

<div class="demo-grid">

<div class="demo-section">
<h3>help — Aide complete</h3>
<p>Toutes les commandes avec flags</p>
<div id="demo-help"></div>
</div>

<div class="demo-section">
<h3>version</h3>
<p>Version et plateforme</p>
<div id="demo-version"></div>
</div>

</div>

<script>
var opts = { cols: 120, rows: 20, autoPlay: false, speed: 1.5, theme: 'monokai', fit: 'width' };
var demos = [
  'ping', 'nslookup', 'traceroute', 'port-check', 'whois', 'subnet',
  'cert-check', 'http-headers', 'ssl-audit', 'dns-enum', 'myip',
  'ps', 'df', 'meminfo', 'sysinfo',
  'secret-scan', 'screenshot', 'base64',
  'query-tables', 'query-env', 'query-interfaces',
  'help', 'version'
];
demos.forEach(function(name) {
  var el = document.getElementById('demo-' + name);
  if (el) {
    AsciinemaPlayer.create('/clientshell-releases/demos/' + name + '.cast', el, opts);
  }
});
</script>
