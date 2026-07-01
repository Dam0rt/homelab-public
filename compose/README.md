# 🐳 Stacks Docker Compose

Quelques stacks représentatives du homelab, **expurgées** — pour montrer *comment* l'infra est déployée, pas seulement ce qui tourne.

| Stack | Hôte | Contenu |
|---|---|---|
| [monitoring](monitoring/README.md) | helium (Pi 4) | Prometheus · Grafana · Loki · Promtail · Unpoller |
| [crowdsec](crowdsec/README.md) | titanium (Unraid) | Moteur · bouncer Cloudflare · dashboard Metabase — IPS comportemental |

**Conventions communes :**

- secrets **externalisés** dans un `.env` (gitignoré), avec un `.env.example` fourni ;
- `restart: unless-stopped`, montages **read-only** quand c'est possible, réseaux dédiés ;
- données expurgées : aucun secret en dur, pas d'adresse MAC, IP privées (RFC1918) uniquement.
