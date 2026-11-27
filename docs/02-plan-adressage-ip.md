# 02 — Plan d'Adressage IP & Segmentation en 12 VLANs

## Principes d'architecture et convention sémantique

Pour remplacer le réseau plat initial (`192.168.1.0/24`), l'infrastructure est segmentée en **12 VLANs** distincts selon une convention d'adressage sémantique où **le 3e octet de chaque préfixe correspond directement à l'identifiant du VLAN** :

- **Réseaux Utilisateurs & Wi-Fi (`192.168.<VLAN_ID>.0`)** : `VLAN 10`, `20`, `30`, `40`, `50` (en `/28`, soit 14 IP utilisables par service, offrant une marge de croissance de 4 à 13 postes) et `VLAN 210` (Wi-Fi Visiteurs en `/26`, soit 62 IP utilisables).
- **Réseaux Infrastructure & Administration (`10.10.<VLAN_ID>.0`)** : `VLAN 99` (Administration), `VLAN 100` (Serveurs), `VLAN 220` (Sauvegarde), `VLAN 230` (Supervision), `VLAN 240` & `245` (liens inter-nœuds Hyper-V en `/30`).
- **Passerelles Virtuelles (VIP)** : portées par le cluster actif/passif **FortiGate 60F (FGCP)** sur la **dernière adresse IP utilisable** de chaque sous-réseau routé (`.14` pour les `/28`, `.62` pour les `/26`, `.6` pour les `/29`).

---

## Table récapitulative des 12 VLANs

| VLAN | Nom & Service | Sous-réseau & Masque | Plage utilisable | Passerelle (VIP) & Broadcast |
| :---: | :--- | :--- | :--- | :--- |
| **`10`** | `VLAN-DIRECTION`<br>Direction | `192.168.10.0/28`<br>(`255.255.255.240`) | **DHCP :** `.1` – `.13` | **GW :** `192.168.10.14`<br>**BC :** `192.168.10.15` |
| **`20`** | `VLAN-RH-COMPTA`<br>RH & Comptabilité | `192.168.20.0/28`<br>(`255.255.255.240`) | **DHCP :** `.1` – `.13` | **GW :** `192.168.20.14`<br>**BC :** `192.168.20.15` |
| **`30`** | `VLAN-COMMERCIAL`<br>Commercial | `192.168.30.0/28`<br>(`255.255.255.240`) | **DHCP :** `.1` – `.13` | **GW :** `192.168.30.14`<br>**BC :** `192.168.30.15` |
| **`40`** | `VLAN-DATA`<br>Data Analyst | `192.168.40.0/28`<br>(`255.255.255.240`) | **DHCP :** `.1` – `.13` | **GW :** `192.168.40.14`<br>**BC :** `192.168.40.15` |
| **`50`** | `VLAN-RD`<br>Recherche & Développement | `192.168.50.0/28`<br>(`255.255.255.240`) | **DHCP :** `.1` – `.13` | **GW :** `192.168.50.14`<br>**BC :** `192.168.50.15` |
| **`99`** | `VLAN-ADMIN`<br>Administration & Management | `10.10.99.0/26`<br>(`255.255.255.192`) | **Statique :** `.1` – `.61` | **GW :** `10.10.99.62`<br>**BC :** `10.10.99.63` |
| **`100`** | `VLAN-SRV`<br>Serveurs de production (VMs) | `10.10.100.0/28`<br>(`255.255.255.240`) | **Statique :** `.1` – `.13` | **GW :** `10.10.100.14`<br>**BC :** `10.10.100.15` |
| **`210`** | `VLAN-WIFI`<br>Wi-Fi Visiteurs (`Datasc-sea-wifi`) | `192.168.210.0/26`<br>(`255.255.255.192`) | **DHCP :** `.1` – `.61` | **GW :** `192.168.210.62`<br>**BC :** `192.168.210.63` |
| **`220`** | `VLAN-BACKUP`<br>Stockage de sauvegarde dédié | `10.10.220.0/29`<br>(`255.255.255.248`) | **Statique :** `.1` – `.5` | **GW :** `10.10.220.6`<br>**BC :** `10.10.220.7` |
| **`230`** | `VLAN-SRV-SUPERVISION`<br>Supervision SI (Centreon) | `10.10.230.0/29`<br>(`255.255.255.248`) | **Statique :** `.1` – `.5` | **GW :** `10.10.230.6`<br>**BC :** `10.10.230.7` |
| **`240`** | `VLAN-CLUSTER-1`<br>Heartbeat & Live Migration Cluster 1 | `10.10.240.0/30`<br>(`255.255.255.252`) | **Statique :** `.1` – `.2` | *Non routé (L2)*<br>**BC :** `10.10.240.3` |
| **`245`** | `VLAN-CLUSTER-2`<br>Heartbeat & Live Migration Cluster 2 | `10.10.245.0/30`<br>(`255.255.255.252`) | **Statique :** `.1` – `.2` | *Non routé (L2)*<br>**BC :** `10.10.245.3` |

> [!NOTE]
> Les postes nomades des collaborateurs connectés au SSID d'entreprise **`CORP-WIFI`** s'authentifient en `WPA3-Enterprise` (`802.1X` / RADIUS) et sont affectés dynamiquement par le serveur RADIUS dans le VLAN métier de leur département (`10`, `20`, `30`, `40` ou `50`).

---

## Affectation des adresses IP statiques par équipement

### VLAN 99 — Administration & Management (`10.10.99.0/26`, GW `10.10.99.62`)

| Équipement / Machine | Type | IP Statique | Rôle |
| :--- | :--- | :---: | :--- |
| `SRV-HyperV 1` (`Dell R350`) | Hôte physique | `10.10.99.1` | Hyperviseur Cluster 1 (management) |
| `SRV-HyperV 2` (`Dell R350`) | Hôte physique | `10.10.99.2` | Hyperviseur Cluster 1 (management) |
| `SRV-HyperV 3` (`Dell R350`) | Hôte physique | `10.10.99.3` | Hyperviseur Cluster 2 (management) |
| `SRV-HyperV 4` (`Dell R350`) | Hôte physique | `10.10.99.4` | Hyperviseur Cluster 2 (management) |
| `Netgear GS752TXv3 (1)` | Switch L2+ | `10.10.99.5` | Commutation cœur (autonome, LACP `20 Gbps`) |
| `Netgear GS752TXv3 (2)` | Switch L2+ | `10.10.99.6` | Commutation cœur (autonome, LACP `20 Gbps`) |
| `APC Smart-UPS X 3000VA (1)` | Onduleur | `10.10.99.7` | Alimentation secourue + carte management |
| `APC Smart-UPS X 3000VA (2)` | Onduleur | `10.10.99.8` | Alimentation secourue + carte management |
| `FortiGate 60F (1)` | Pare-feu | `10.10.99.9` | Nœud 1 cluster HA (FGCP) |
| `FortiGate 60F (2)` | Pare-feu | `10.10.99.10` | Nœud 2 cluster HA (FGCP) |
| `HPE Aruba AP-505 (1)` | Borne Wi-Fi | `10.10.99.11` | Point d'accès Wi-Fi 6 (`CORP-WIFI` / Visiteurs) |
| `HPE Aruba AP-505 (2)` | Borne Wi-Fi | `10.10.99.12` | Point d'accès Wi-Fi 6 (`CORP-WIFI` / Visiteurs) |
| `PC Administrateur` | Poste admin | `10.10.99.61` | Unique poste autorisé pour les flux d'administration |

### VLAN 100 — Serveurs de production (`10.10.100.0/28`, GW `10.10.100.14`)

| Équipement / Machine | Type | IP Statique | Rôle |
| :--- | :--- | :---: | :--- |
| `SRV-AD 1` | VM (Cluster 1) | `10.10.100.1` | Contrôleur de domaine principal (`AD DS` / `DNS`) |
| `SRV-COMPTA` | VM (Cluster 1) | `10.10.100.2` | Serveur de comptabilité (`Sage 100c`) |
| `SRV-AD 2` | VM (Cluster 2) | `10.10.100.3` | Contrôleur secondaire (`AD DS` / `DNS` / `RADIUS NPS`) |
| `SRV-APP` | VM (Cluster 2) | `10.10.100.4` | Base de données métier (`SQL Server 2022`) |
| `SRV-VEEAM` | VM (Cluster 2) | `10.10.100.5` | Console & orchestration `Veeam Backup & Replication` |

### VLANs dédiés : Backup (`220`), Supervision (`230`) & Clusters (`240` / `245`)

| VLAN | Équipement / Machine | IP Statique | Passerelle | Rôle |
| :---: | :--- | :---: | :---: | :--- |
| **`220`** | `Synology RS2423+` | `10.10.220.1` | `10.10.220.6` | Dépôt local de sauvegardes (`4×4 To` RAID 10) |
| **`230`** | `SRV-SUPERVISION` | `10.10.230.1` | `10.10.230.6` | Monitoring `Centreon 24.10 LTS` (`Ubuntu 24.04 LTS`) |
| **`240`** | `SRV-HyperV 1` (NIC Cluster) | `10.10.240.1` | *Non routé* | Lien dédié Cluster 1 (Heartbeat / Live Migration) |
| **`240`** | `SRV-HyperV 2` (NIC Cluster) | `10.10.240.2` | *Non routé* | Lien dédié Cluster 1 (Heartbeat / Live Migration) |
| **`245`** | `SRV-HyperV 3` (NIC Cluster) | `10.10.245.1` | *Non routé* | Lien dédié Cluster 2 (Heartbeat / Live Migration) |
| **`245`** | `SRV-HyperV 4` (NIC Cluster) | `10.10.245.2` | *Non routé* | Lien dédié Cluster 2 (Heartbeat / Live Migration) |
