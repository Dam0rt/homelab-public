# 📊 Stack monitoring — HELIUM

Observabilité du homelab (**métriques + logs**), en Docker Compose sur **helium** (Raspberry Pi 4) — volontairement hors des hôtes supervisés.

> Vue d'ensemble & captures des dashboards : **[../../supervision.md](../../supervision.md)**.

## Composants

| Service | Image | Port | Rôle |
|---|---|---|---|
| **Prometheus** | `prom/prometheus` | 9090 | base de métriques (rétention 30 j) |
| **Grafana** | `grafana/grafana` | 3000 | tableaux de bord |
| **Loki** | `grafana/loki` | 3100 | logs centralisés (rétention 30 j, purge auto) |
| **Promtail** | `grafana/promtail` | 9080 · 1514 | collecte des logs + réception Syslog |
| **node-exporter** | `prom/node-exporter` | — | métriques de l'hôte (aucun port exposé) |
| **Unpoller** | `ghcr.io/unpoller/unpoller` | 9130 | export des métriques UniFi |

## Ce qui est collecté

- **Métriques** : l'hôte helium, `titanium` (Unraid), `uranium` (Proxmox via l'exporter PVE), et le contrôleur UniFi (Unpoller).
- **Logs** : Promtail récupère les logs de l'hôte et des conteneurs, et fait aussi **office de serveur Syslog** pour recevoir les journaux de l'UDM. Rétention 30 jours, purge automatique via le `compactor` de Loki.

## Secrets

Les identifiants Unpoller ne sont **pas** dans le `docker-compose.yml` : ils viennent d'un fichier `.env` (gitignoré).

```bash
cp .env.example .env
# renseigne un utilisateur UniFi READ-ONLY dédié dans .env
```

## Déploiement

```bash
docker compose up -d
```

Grafana est ensuite accessible sur `http://<hôte>:3000`.

## Fichiers

- [`docker-compose.yml`](docker-compose.yml) — orchestration des 6 conteneurs
- [`prometheus.yml`](prometheus.yml) — cibles de collecte des métriques
- [`loki-config.yaml`](loki-config.yaml) — stockage, rétention, compactor
- [`promtail-config.yaml`](promtail-config.yaml) — jobs de collecte + serveur Syslog
- [`.env.example`](.env.example) — modèle des secrets (à copier en `.env`)
