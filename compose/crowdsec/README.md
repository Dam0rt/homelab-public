# 🛡️ CrowdSec, IPS comportemental

Détection comportementale et bannissement au bord, déployés sur titanium (Unraid). Trois conteneurs :

| Conteneur | Rôle |
|---|---|
| **crowdsec** | moteur : parse les logs, applique les scénarios, prend les décisions (LAPI) |
| **crowdsec-cloudflare-bouncer** | pousse les IP bannies vers une liste Cloudflare (blocage avant l'origine) |
| **crowdsec-dashboard** (Metabase) | visualisation des alertes et décisions |

## Le pipeline

```
Logs (SSH / syslog / reverse proxy Zoraxy)
   → moteur CrowdSec (scénarios) → décision de ban
   → bouncer Cloudflare → IP bloquée AU BORD (avant d'atteindre le homelab)
```

- **Sources de logs** : `auth.log` (brute-force SSH), `syslog`, et les logs du reverse proxy Zoraxy (services exposés).
- **Collections** : `crowdsecurity/linux`, `nginx`, `http-cve` (scénarios d'attaques web courantes).
- **Blocage en amont** : le bouncer synchronise les bans avec Cloudflare ; le trafic malveillant n'atteint jamais le tunnel ni l'origine.

## Secrets

- Le jeton API Cloudflare du bouncer vit dans la config CrowdSec montée (`/etc/crowdsec`, sur l'hôte), jamais dans ce dépôt.
- Aucun secret dans le compose : tout passe par des volumes / config hors-repo.

## Déploiement

```bash
docker compose up -d
```
