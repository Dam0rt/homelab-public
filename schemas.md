# 🗺️ Schémas

Diagrammes **Mermaid** (rendu natif sur GitHub) — la vue **logique** de l'infra. Des versions **isométriques** (FossFLOW) illustrent les pages dédiées.

## 1. Topologie physique

```mermaid
graph TD
    INET["🌐 Internet — fibre, IP publique fixe"]
    BOX["📦 Box FAI — mode DMZ"]
    UDM["🛡️ UDM Pro — pare-feu / contrôleur"]
    SW["🔀 Switch 16 PoE"]
    AP["📶 Point d'accès WiFi"]
    TIT["🗄️ titanium — Unraid (NAS + Docker + VM)"]
    URA["🖥️ uranium — Proxmox (VM/LXC)"]
    HEL["🍓 helium — Pi4 (supervision)"]
    CAM["📹 Caméras (CCTV)"]
    CLI["💻 Postes & clients WiFi"]

    INET --> BOX
    BOX --> UDM
    UDM -->|SFP+| SW
    SW --> TIT
    SW --> URA
    SW --> HEL
    SW --> CAM
    SW --> AP
    AP -. WiFi .-> CLI
```

## 2. Segmentation (zones de confiance)

```mermaid
graph LR
    EXT["🌐 Internet"]
    INT["🔐 Internal — MNGT · USERS · SRVR · CTNR · DNS · LABS · VOIP"]
    IOT["🔌 IoT — objets · multimédia · caméras"]
    GUEST["👥 Invité"]
    DMZ["🟠 DMZ — reverse proxy"]
    VPN["🔑 VPN — WireGuard"]
    SUP["📊 Supervision"]

    INT -->|autorisé| EXT
    IOT -->|autorisé| EXT
    GUEST -->|autorisé| EXT
    DMZ -->|autorisé| EXT
    IOT -.->|❌ bloqué| INT
    GUEST -.->|❌ isolé| INT
    DMZ -.->|❌ sens unique| INT
    VPN -->|✅ admin| INT
    SUP -->|✅ scrape ciblé| INT
```

## 3. Flux d'accès (exposition Internet)

```mermaid
flowchart TD
    V["👤 Visiteur Internet"] --> CF["☁️ Cloudflare — WAF + CrowdSec + Tunnel"]
    TUN["cloudflared (sortant)"] -. tunnel .-> CF
    CF --> ZX["🔀 Reverse proxy (DMZ)"]
    ZX --> PUB["🌍 Services publics — blog · vidéos · demandes média"]
    ZX -. split-DNS interne .-> LAN["🏠 Accès interne direct"]
```

## 4. Sauvegarde (3-2-1)

```mermaid
flowchart LR
    SRC["📦 Sources — configs · VM · données"]
    HUB[("💽 NAS — copie locale + parité")]
    OFF[("☁️ Cloud souverain UE — offsite chiffré")]
    SRC --> HUB
    HUB ==>|restic : chiffre + déduplique| OFF
```

---

*Détails : [réseau](network.md) · [segmentation VLAN](vlans.md) · [sauvegarde](backup.md).*
