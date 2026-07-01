# 🌐 Réseau — WAN, DNS, certificats & accès distant

## Connexion Internet (WAN)

```
Internet (fibre, IP publique fixe) → Box FAI (mode DMZ) → UDM Pro → réseau segmenté
```

La **box FAI est en mode DMZ** vers l'UDM Pro : elle pousse **tout** le trafic entrant vers l'UDM, **sans aucune redirection de port** sur la box. → Tout le filtrage et le routage se font sur l'**UDM Pro** — **un seul point de contrôle**, pas de règles éparpillées sur la box.

## DNS — filtrant et forcé

- **unbound** : résolveur **récursif** local (pas de DNS tiers à qui faire confiance).
- **AdGuard Home** : **filtrage** en amont (blocklists anti-pub / tracking / malware).
- **DNS forcé** : une règle pare-feu **redirige tout le trafic DNS (port 53)** vers AdGuard → un appareil (ou un malware) **ne peut pas contourner** le filtrage en pointant un DNS externe.

→ Filtrage **à l'échelle du réseau**, y compris sur les appareils qui ne le respecteraient pas seuls (IoT, TV…).

## Certificats & accès interne

- **Certificats wildcard Let's Encrypt** (`*.exemple.tld`) → **HTTPS valide** sur **tous** les services internes, zéro avertissement de certificat.
- **Reverse proxy (Zoraxy)** en DMZ : point d'entrée HTTPS unique.
- **Split-DNS** : un même nom (`service.exemple.tld`) résout **en interne** vers le reverse proxy local, **en externe** vers Cloudflare → mêmes URLs partout, sans exposer l'interne.

## Accès distant

| Méthode | Usage |
|---|---|
| **WireGuard** (VPN) | accès **admin** distant chiffré → on entre dans le réseau interne comme si on était sur le LAN |
| **Cloudflare Tunnel** (cloudflared) | expose les **services publics** **sans ouvrir aucun port** sur la box |
| **Cloudflare WAF** | filtrage applicatif devant les services exposés |

## Modèle d'exposition

- **Aucun port ouvert** sur la box vers Internet — tout passe par **Cloudflare Tunnel**.
- **Public** (via Cloudflare + WAF) : seulement quelques services choisis (blog, vidéos, demandes média).
- **Interne** (LAN + VPN) : tout le reste — dont le **coffre-fort de mots de passe**, jamais exposé.
- **Le moins de surface exposée possible** : aucun port ouvert, une poignée de services publics seulement, et le reverse proxy isolé du reste (DMZ).

---

*Segmentation détaillée : [vlans.md](vlans.md) · audit du pare-feu : [security.md](security.md).*
