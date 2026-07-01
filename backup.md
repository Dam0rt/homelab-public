# 💾 Sauvegarde & restauration

Stratégie **3-2-1** : **3** copies, **2** supports, **1** hors-site chiffré. Principe directeur : *« un backup jamais restauré n'existe pas »*.

## Le 3-2-1 en pratique

| | Quoi | Où |
|---|---|---|
| **Copie locale** | données + configs des services | NAS (Unraid) — array protégé par **parité** |
| **2ᵉ support** | sauvegardes appdata / VM / config | jeu de sauvegardes séparé sur le NAS |
| **Hors-site** | l'irremplaçable, **chiffré** | **cloud souverain (UE)** via **restic** |

- **Chiffrement côté client** (restic, AES-256) → le fournisseur ne voit que du chiffré, jamais mes fichiers.
- **Hébergeur européen** plutôt qu'un géant américain → je préfère savoir où vivent mes données.
- **Déduplication + versionnement** → historique profond pour peu de volume.

## Sauvegarde des VM / conteneurs (Proxmox)

- **vzdump** (snapshot + `fsfreeze` → cohérent même pour les bases **SQLite**) → stockage **NFS** sur le NAS → repris automatiquement dans le push hors-site.
- Rétention échelonnée (quotidienne / hebdomadaire / mensuelle).

## Restauration **testée** (pas seulement configurée)

Un backup ne « compte » que s'il se restaure :
- Restauration d'un conteneur critique dans un **environnement jetable** → vérification que les **données et les clés sont intactes** → destruction.
- Contrôle que la donnée hors-site est bien **présente et déchiffrable**.

## Fiabilisation & supervision

En vérifiant la chaîne, je me suis aperçu qu'une sauvegarde échouait depuis plusieurs jours sans alerte — un montage réseau tombé en « stale ».

- Cause racine corrigée : bascule du montage **CIFS → NFS** (reconnexion automatique après coupure).
- **Alerte sur échec** ajoutée : notification push sur le téléphone via un serveur de notification auto-hébergé.

Depuis, un échec de sauvegarde déclenche une notification au lieu de passer inaperçu dans un log.

![Notification de sauvegarde Proxmox reçue sur le téléphone](images/backup-notification.png)

*Chaque sauvegarde Proxmox notifie sur le téléphone — succès comme échec — via le serveur de notification auto-hébergé.*

---

*Supervision & alerting : [services.md](services.md).*
