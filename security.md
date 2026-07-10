# 🛡️ Sécurité

L'idée : empiler quelques couches et surtout vérifier de temps en temps que ça tient vraiment, au lieu de partir du principe que « ça marche ».

## Posture

| Couche | Mise en œuvre |
|---|---|
| **Segmentation** | pare-feu zone-based (8 zones), VLANs cloisonnés (détail : [vlans.md](vlans.md)) |
| **Filtrage DNS** | AdGuard + unbound, DNS forcé, contournement impossible (détail : [network.md](network.md)) |
| **Exposition** | aucun port ouvert (Cloudflare Tunnel + WAF), reverse proxy en DMZ |
| **IDS / IPS** | threat management intégré au UDM Pro (détection / prévention d'intrusion) |
| **IPS applicatif** | CrowdSec : analyse des logs des services exposés, bannissement des IP malveillantes via bouncer Cloudflare |
| **Accès admin** | VPN WireGuard obligatoire, SSH par clé FIDO2 (mot de passe désactivé, port non standard), management restreint (whitelist) |
| **Secrets** | coffre Vaultwarden durci (Argon2id, 2FA matérielle FIDO2) |

## Durcissement des accès

SSH n'accepte que les clés (FIDO2 / YubiKey) : mot de passe désactivé, port non standard. La même MFA matérielle protège le coffre Vaultwarden et le contrôleur UniFi. L'accès distant passe uniquement par WireGuard, qui fait entrer dans le réseau comme si on était sur le LAN ; rien n'est exposé en direct. Enfin, seules les sources admin (whitelist) atteignent les plans d'administration.

## Défense des services exposés

Les quelques services publics passent par Cloudflare (Tunnel, rien d'ouvert sur le WAN) avec un WAF réglé : géo-filtrage, *challenges* sur les pages sensibles (connexion), blocage des sondes d'exploit courantes. En complément, CrowdSec analyse les logs du reverse proxy et des services exposés, puis pousse les IP malveillantes vers un bouncer Cloudflare : elles sont bloquées chez Cloudflare, avant même d'atteindre l'origine.

## Mises à jour

Les machines Debian et le Pi se mettent à jour par `apt update && upgrade` ; Unraid et Proxmox signalent les leurs. Côté conteneurs, l'outil natif d'Unraid couvre le cœur et Dockge les stacks Docker Compose. Les services non critiques suivent, sans urgence.

> Chantier en cours : migration de l'ensemble en Docker Compose pour des mises à jour versionnées et reproductibles ([compose/](compose/README.md)).

## Audit du pare-feu

Revue de l'ensemble des règles (zones, matrice inter-zones, politiques) contre une grille de bonnes pratiques : segmentation, exposition, moindre privilège, hygiène des règles.

Comment le score est obtenu :

1. Export en lecture seule de la configuration live du contrôleur : règles, zones, groupes d'adresses, redirections NAT.
2. Passage sur une grille de contrôle par familles : segmentation (isolation des VLANs sensibles : IoT, invités, caméras), exposition Internet (redirections NAT, règles WAN entrantes), moindre privilège (règles trop larges type « ANY → ANY »), hygiène (règles mortes, doublons, objets orphelins).
3. Chaque écart retire des points selon sa criticité (critique / important / mineur), en partant de 100.
4. Contre-vérification manuelle de chaque écart signalé : plusieurs « manques » remontés par l'audit outillé étaient en réalité couverts par une règle prédéfinie ou par la matrice de zones, donc écartés comme faux positifs après vérification, pas ignorés.

L'audit est outillé (analyse assistée par IA de l'export de configuration) puis arbitré à la main.

Résultat sur trois passes : de 74/100 au premier audit à **~89/100** (healthy) après corrections. Principaux écarts corrigés au fil des passes :

- Le réseau de management n'était pas restreint aux seules sources admin : fermé depuis (whitelist).
- Surface entrante réduite à un seul port (fermeture du port 80, suppression d'une redirection d'un serveur de jeu obsolète).
- Forçage DNS (impossible de contourner le résolveur filtrant) et nettoyage des règles mortes.

## Modèle de menace

Priorité : empêcher la compromission du réseau et des serveurs (intrusion, ransomware, propagation latérale). La segmentation et la matrice de zones empêchent un segment compromis, IoT ou invité, de pivoter vers les serveurs. La surface publique se réduit à quelques services derrière WAF. Si malgré tout un ransomware passait, la sauvegarde 3-2-1 et sa copie hors-site chiffrée et versionnée servent de dernier rempart ([backup.md](backup.md)). Physiquement, les équipements réseau sont en baie verrouillée, complétés par alarme, détection incendie et vidéosurveillance.

*Un homelab n'est jamais vraiment « fini » : je le durcis au fil de l'eau, sans prétendre que tout est parfait.*

## Gestion des secrets

Les mots de passe vivent dans un coffre Vaultwarden auto-hébergé, durci (2FA matérielle YubiKey / FIDO2, KDF Argon2id), sauvegardé chiffré et dont la restauration a été testée ([backup.md](backup.md)). Des clés FIDO2 de secours sont conservées hors-ligne, dont une en coffre ignifuge, pour ne pas rester bloqué si la principale disparaît. Côté dépôt, un `.gitignore` protège les fichiers de config sensibles et la version publique est nettoyée de tout ce qui identifie l'installation (IP publique, MAC, domaine, clés).
