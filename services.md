# 🧩 Services auto-hébergés

Une vingtaine de services, répartis entre **Docker (Unraid)**, **VM/LXC (Proxmox)** et le **Pi (supervision)**. L'exposition Internet est **sélective** : seule une poignée passe par le reverse proxy + Cloudflare, **tout le reste est interne** (accès LAN ou VPN).

## 🎬 Médias & divertissement
| Service | Rôle |
|---|---|
| **Jellyfin** | serveur multimédia |
| **Jellystat** | statistiques de visionnage |
| **PeerTube** | vidéos auto-hébergées *(exposé)* |

## 🏠 Domotique & productivité
| Service | Rôle |
|---|---|
| **Home Assistant** | domotique (VM dédiée) |
| **Vaultwarden** | coffre-fort de mots de passe *(interne uniquement)* |
| **Paperless** | GED / archivage de documents |
| **WordPress** | blog à l'apex du domaine *(exposé)* |
| **FreshRSS** | agrégateur RSS |
| **n8n** | automatisation / workflows |
| **Wallos · ByteStash · Spoolman** | abonnements · snippets · bobines d'impression 3D |

## 🛡️ Infrastructure & sécurité
| Service | Rôle |
|---|---|
| **AdGuard Home + unbound** | DNS filtrant + résolveur récursif |
| **Zoraxy** | reverse proxy (point d'entrée HTTPS) |
| **cloudflared** | Cloudflare Tunnel (exposition sans port ouvert) |
| **WireGuard** | VPN d'accès distant |
| **CrowdSec** | IPS comportemental — bannissement des IP malveillantes via bouncer Cloudflare |

## 📊 Supervision & observabilité
| Service | Rôle |
|---|---|
| **Prometheus + Grafana + Loki** | métriques · dashboards · logs (sur le Pi, hors SPOF) |
| **Uptime Kuma** | disponibilité + alerting (Bark / SMS) |

## 🗄️ Données
| Service | Rôle |
|---|---|
| **MariaDB · PostgreSQL · Redis** | bases partagées (Ghost, Jellystat, PeerTube…) |

## 🧪 Gestion & labo
| Service | Rôle |
|---|---|
| **GLPI** | gestion de parc / ticketing (ITIL) |
| **Dockge** | gestion des stacks Docker |
| **Kali · EVE-NG** | labo offensif · labo réseau |

---

## Modèle d'exposition

- **Interne uniquement** (LAN + VPN) : la grande majorité — dont **Vaultwarden** (coffre, **jamais exposé**).
- **Exposé** via **Cloudflare Tunnel + reverse proxy** : seulement quelques services publics (blog, demandes média, vidéos) — derrière le **WAF Cloudflare**, **sans aucun port ouvert** sur la box.
- **Split-DNS** : un même nom résout en **interne** vers le reverse proxy local, en **externe** vers Cloudflare → surface d'exposition minimale.

→ Détail réseau & exposition : **[network.md](network.md)**.

---

## Supervision

Tous ces services sont supervisés (disponibilité, métriques, logs) et un échec déclenche une alerte → **[supervision.md](supervision.md)** (dashboards Grafana + Uptime Kuma).
