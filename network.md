# 🌐 Réseau : WAN, DNS, certificats & accès distant

## Connexion Internet (WAN)

```
Internet (fibre, IP publique fixe) → Box FAI (mode DMZ) → UDM Pro → réseau segmenté
```

La box FAI est en mode DMZ vers l'UDM Pro : elle pousse tout le trafic entrant vers l'UDM, sans aucune redirection de port sur la box. Tout le filtrage et le routage se font donc sur l'UDM Pro, un seul point de contrôle plutôt que des règles éparpillées sur la box.

## DNS filtrant et forcé

La résolution est faite en local par unbound, un résolveur récursif : pas de DNS tiers à qui faire confiance. AdGuard Home filtre en amont avec des blocklists (pub, tracking, malware). Et pour qu'aucun appareil ne passe à côté, une règle pare-feu redirige tout le trafic DNS (port 53) vers AdGuard : même un malware qui pointe un DNS externe se fait rattraper. Le filtrage vaut ainsi à l'échelle du réseau, y compris sur les appareils qui ne le respecteraient pas seuls (IoT, TV…).

## Certificats & accès interne

Un certificat wildcard Let's Encrypt (`*.exemple.tld`) donne un HTTPS valide sur tous les services internes, sans avertissement de navigateur. Le point d'entrée est unique : le reverse proxy Zoraxy, en DMZ. Le split-DNS complète le tout : un même nom (`service.exemple.tld`) résout en interne vers le reverse proxy local et en externe vers Cloudflare, donc les mêmes URLs fonctionnent partout sans exposer l'interne.

## Accès distant

| Méthode | Usage |
|---|---|
| **WireGuard** (VPN) | accès admin distant chiffré : on entre dans le réseau interne comme si on était sur le LAN |
| **Cloudflare Tunnel** (cloudflared) | expose les services publics sans ouvrir aucun port sur la box |
| **Cloudflare WAF** | filtrage applicatif devant les services exposés |

## Modèle d'exposition

Rien n'écoute côté WAN : tout le trafic entrant passe par Cloudflare Tunnel, derrière le WAF. Seuls quelques services choisis sont publics (blog, vidéos, demandes média) ; tout le reste, dont le coffre-fort de mots de passe, n'est accessible qu'en LAN ou VPN. Quant au reverse proxy, seul composant joignable de l'extérieur, il vit isolé du reste en DMZ. Le découpage est dans [vlans.md](vlans.md), et l'audit de ces règles dans [security.md](security.md).
