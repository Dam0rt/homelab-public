# 🛡️ Sécurité

L'idée : empiler quelques couches (segmenter, filtrer, exposer le moins possible) et surtout **vérifier** de temps en temps que ça tient vraiment, au lieu de partir du principe que « ça marche ».

## Posture

| Couche | Mise en œuvre |
|---|---|
| **Segmentation** | pare-feu **zone-based** (8 zones), VLANs cloisonnés → [vlans.md](vlans.md) |
| **Filtrage DNS** | AdGuard + unbound, **DNS forcé** (contournement impossible) → [network.md](network.md) |
| **Exposition** | **aucun port ouvert** (Cloudflare Tunnel + WAF), reverse proxy en DMZ |
| **IDS / IPS** | threat management intégré au **UDM Pro** (détection / prévention d'intrusion) |
| **IPS applicatif** | **CrowdSec** — analyse des logs des services exposés → bannissement des IP malveillantes via **bouncer Cloudflare** |
| **Accès admin** | **VPN WireGuard** obligatoire · **SSH par clé FIDO2** (mot de passe désactivé, port non-standard) · management restreint (whitelist) |
| **Secrets** | coffre **Vaultwarden** durci (Argon2id, **2FA matérielle FIDO2**) |

## Durcissement des accès

- **SSH** : authentification **par clé (FIDO2 / YubiKey) uniquement**, mot de passe désactivé, **port non-standard**.
- **MFA matérielle (FIDO2)** sur le coffre **Vaultwarden** et le contrôleur **UniFi**.
- **Accès distant uniquement par WireGuard** : on entre dans le réseau interne comme sur le LAN, rien n'est exposé en direct.
- **Management restreint** : seules les sources admin (whitelist) atteignent les plans d'administration.

## Défense des services exposés

Les quelques services publics passent par **Cloudflare** (Tunnel, aucun port ouvert) avec un **WAF** réglé : géo-filtrage, *challenges* sur les pages sensibles (connexion), blocage des sondes d'exploit courantes. En complément, **CrowdSec** analyse les logs (reverse proxy + services exposés) et pousse les IP malveillantes vers un **bouncer Cloudflare** → bannissement **au bord**, avant même d'atteindre l'origine.

## Mises à jour

- **Hôtes** : `apt update && upgrade` sur les machines Debian / Pi, suivi des notifications de mise à jour Unraid & Proxmox.
- **Conteneurs** : outil de mise à jour natif (Unraid) pour les conteneurs cœur, **Dockge** pour les stacks Docker Compose.
- Services non critiques → mises à jour appliquées au fil de l'eau.

> Chantier en cours : **migration de l'ensemble en Docker Compose** pour des mises à jour versionnées et reproductibles → [compose/](compose/README.md).

## Audit du pare-feu

Revue de l'ensemble des règles (zones, matrice inter-zones, politiques) contre une grille de bonnes pratiques : segmentation, exposition, moindre privilège, hygiène des règles.

**Comment le score est obtenu :**

1. **Export en lecture seule de la configuration live** du contrôleur : règles, zones, groupes d'adresses, redirections NAT.
2. **Passage sur une grille de contrôle** par familles — segmentation (isolation des VLANs sensibles : IoT, invités, caméras), exposition Internet (redirections NAT, règles WAN entrantes), moindre privilège (règles trop larges type « ANY → ANY »), hygiène (règles mortes, doublons, objets orphelins).
3. Chaque écart retire des points selon sa criticité (critique / important / mineur), en partant de 100.
4. **Contre-vérification manuelle de chaque écart signalé** : plusieurs « manques » remontés par l'audit outillé étaient en réalité couverts par une règle prédéfinie ou par la matrice de zones → écartés comme faux positifs après vérification, pas ignorés.

L'audit est **outillé** (analyse assistée par IA de l'export de configuration) puis **arbitré à la main** — l'outil trouve vite, l'humain tranche.

**Résultat sur trois passes : 74/100 au premier audit → ~89/100 (healthy)** après corrections. Principaux écarts corrigés au fil des passes :

- Le réseau de **management** n'était pas restreint aux seules sources admin → fermé (whitelist).
- Surface entrante réduite à **un seul port** (fermeture du port 80, suppression d'une redirection de jeu obsolète).
- **Forçage DNS** (impossible de contourner le résolveur filtrant) et nettoyage des règles mortes.

## Modèle de menace

Priorité : empêcher la **compromission du réseau et des serveurs** — intrusion, **ransomware**, propagation latérale.

- **Réseau** : segmentation VLAN + matrice de zones → un segment compromis (IoT, invité) **ne peut pas pivoter** vers les serveurs.
- **Exposition** : aucun port ouvert, surface publique réduite à quelques services derrière WAF.
- **Données** : sauvegarde **3-2-1** avec copie hors-site **chiffrée et versionnée** → dernier rempart anti-ransomware ([backup.md](backup.md)).
- **Physique** : équipements réseau en **baie verrouillée**, complétés par alarme, détection incendie et vidéosurveillance.

*Un homelab n'est jamais vraiment « fini » : je le durcis au fil de l'eau, sans prétendre que tout est parfait.*

## Gestion des secrets

- **Coffre auto-hébergé** (Vaultwarden) — **2FA matérielle** (YubiKey / FIDO2), KDF **Argon2id**.
- **Aucun secret en clair** publié : un `.gitignore` protège les fichiers de config sensibles ; tout est **expurgé** de la version publique (IP publique, MAC, domaine, clés).
- **Clés FIDO2 de secours** conservées **hors-ligne** (dont une en coffre ignifuge) → résilience en cas de perte de la clé principale.
- Coffre **sauvegardé, chiffré, restauration testée** → [backup.md](backup.md).

---

*Segmentation : [vlans.md](vlans.md) · réseau & exposition : [network.md](network.md).*
