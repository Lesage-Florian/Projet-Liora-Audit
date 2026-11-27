# 02 — Plan d'Adressage IP & Segmentation en 12 VLANs

## Principes d'architecture et convention sémantique

Pour remplacer le réseau plat initial (`192.168.1.0/24`), l'infrastructure est segmentée en **12 VLANs** distincts selon une convention d'adressage sémantique où **le 3e octet de chaque préfixe correspond directement à l'identifiant du VLAN** :

- **Réseaux Utilisateurs & Wi-Fi (`192.168.<VLAN_ID>.0`)** : `VLAN 10`, `20`, `30`, `40`, `50` (en `/28`, soit 14 IP utilisables par service, offrant une marge de croissance de 4 à 13 postes) et `VLAN 210` (Wi-Fi Visiteurs en `/26`, soit 62 IP utilisables).
- **Réseaux Infrastructure & Administration (`10.10.<VLAN_ID>.0`)** : `VLAN 99` (Administration), `VLAN 100` (Serveurs), `VLAN 220` (Sauvegarde), `VLAN 230` (Supervision), `VLAN 240` & `245` (liens inter-nœuds Hyper-V en `/30`).
- **Passerelles Virtuelles (VIP)** : portées par le cluster actif/passif **FortiGate 60F (FGCP)** sur la **dernière adresse IP utilisable** de chaque sous-réseau routé (`.14` pour les `/28`, `.62` pour les `/26`, `.6` pour les `/29`).

---

## Table récapitulative des 12 VLANs

| VLAN ID | Nom du VLAN | Service / Rôle | Adresse Réseau | Masque / CIDR | Plage IP Statique | Plage DHCP | Passerelle (VIP) | Broadcast |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| **`10`** | `VLAN-DIRECTION` | Direction | `192.168.10.0` | `255.255.255.240` (`/28`) | — | `192.168.10.1` – `192.168.10.13` | `192.168.10.14` | `192.168.10.15` |
| **`20`** | `VLAN-RH-COMPTA` | RH & Comptabilité | `192.168.20.0` | `255.255.255.240` (`/28`) | — | `192.168.20.1` – `192.168.20.13` | `192.168.20.14` | `192.168.20.15` |
| **`30`** | `VLAN-COMMERCIAL` | Commercial | `192.168.30.0` | `255.255.255.240` (`/28`) | — | `192.168.30.1` – `192.168.30.13` | `192.168.30.14` | `192.168.30.15` |
| **`40`** | `VLAN-DATA` | Data Analyst | `192.168.40.0` | `255.255.255.240` (`/28`) | — | `192.168.40.1` – `192.168.40.13` | `192.168.40.14` | `192.168.40.15` |
| **`50`** | `VLAN-RD` | Recherche & Développement | `192.168.50.0` | `255.255.255.240` (`/28`) | — | `192.168.50.1` – `192.168.50.13` | `192.168.50.14` | `192.168.50.15` |
| **`99`** | `VLAN-ADMIN` | Administration & Management | `10.10.99.0` | `255.255.255.192` (`/26`) | `10.10.99.1` – `10.10.99.61` | — | `10.10.99.62` | `10.10.99.63` |
| **`100`** | `VLAN-SRV` | Serveurs de production (VMs) | `10.10.100.0` | `255.255.255.240` (`/28`) | `10.10.100.1` – `10.10.100.13` | — | `10.10.100.14` | `10.10.100.15` |
| **`210`** | `VLAN-WIFI` | Wi-Fi Visiteurs (`Datasc-sea-wifi`) | `192.168.210.0` | `255.255.255.192` (`/26`) | — | `192.168.210.1` – `192.168.210.61` | `192.168.210.62` | `192.168.210.63` |
| **`220`** | `VLAN-BACKUP` | Stockage de sauvegarde dédié | `10.10.220.0` | `255.255.255.248` (`/29`) | `10.10.220.1` – `10.10.220.5` | — | `10.10.220.6` | `10.10.220.7` |
| **`230`** | `VLAN-SRV-SUPERVISION` | Supervision SI (Centreon) | `10.10.230.0` | `255.255.255.248` (`/29`) | `10.10.230.1` – `10.10.230.5` | — | `10.10.230.6` | `10.10.230.7` |
| **`240`** | `VLAN-CLUSTER-1` | Heartbeat & Live Migration Cluster 1 | `10.10.240.0` | `255.255.255.252` (`/30`) | `10.10.240.1` – `10.10.240.2` | — | *Non routé (L2)* | `10.10.240.3` |
| **`245`** | `VLAN-CLUSTER-2` | Heartbeat & Live Migration Cluster 2 | `10.10.245.0` | `255.255.255.252` (`/30`) | `10.10.245.1` – `10.10.245.2` | — | *Non routé (L2)* | `10.10.245.3` |

> [!NOTE]
> Les postes nomades des collaborateurs connectés au SSID d'entreprise **`CORP-WIFI`** s'authentifient en `WPA3-Enterprise` (`802.1X` / RADIUS) et sont affectés dynamiquement par le serveur RADIUS dans le VLAN métier de leur département (`10`, `20`, `30`, `40` ou `50`).

---

## Affectation des adresses IP statiques par équipement

| VLAN ID | Équipement / Machine | Type | Adresse IP Statique | Masque / CIDR | Passerelle | Rôle |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- |
| **`99`** | `SRV-HyperV 1` (Dell PowerEdge R350) | Hôte physique | `10.10.99.1` | `/26` | `10.10.99.62` | Hyperviseur Cluster 1 (management) |
| **`99`** | `SRV-HyperV 2` (Dell PowerEdge R350) | Hôte physique | `10.10.99.2` | `/26` | `10.10.99.62` | Hyperviseur Cluster 1 (management) |
| **`99`** | `SRV-HyperV 3` (Dell PowerEdge R350) | Hôte physique | `10.10.99.3` | `/26` | `10.10.99.62` | Hyperviseur Cluster 2 (management) |
| **`99`** | `SRV-HyperV 4` (Dell PowerEdge R350) | Hôte physique | `10.10.99.4` | `/26` | `10.10.99.62` | Hyperviseur Cluster 2 (management) |
| **`99`** | `Switch Netgear GS752TXv3 (1)` | Switch L2+ | `10.10.99.5` | `/26` | `10.10.99.62` | Commutation cœur (autonome, LACP `20 Gbps`) |
| **`99`** | `Switch Netgear GS752TXv3 (2)` | Switch L2+ | `10.10.99.6` | `/26` | `10.10.99.62` | Commutation cœur (autonome, LACP `20 Gbps`) |
| **`99`** | `Smart-UPS X 3000VA (1)` | Onduleur | `10.10.99.7` | `/26` | `10.10.99.62` | Alimentation secourue + carte management |
| **`99`** | `Smart-UPS X 3000VA (2)` | Onduleur | `10.10.99.8` | `/26` | `10.10.99.62` | Alimentation secourue + carte management |
| **`99`** | `FortiGate 60F (1)` | Pare-feu | `10.10.99.9` | `/26` | `10.10.99.62` | Nœud 1 cluster HA (FGCP) |
| **`99`** | `FortiGate 60F (2)` | Pare-feu | `10.10.99.10` | `/26` | `10.10.99.62` | Nœud 2 cluster HA (FGCP) |
| **`99`** | `HPE Aruba AP-505 (1)` | Borne Wi-Fi | `10.10.99.11` | `/26` | `10.10.99.62` | Point d'accès Wi-Fi 6 (`CORP-WIFI` / Visiteurs) |
| **`99`** | `HPE Aruba AP-505 (2)` | Borne Wi-Fi | `10.10.99.12` | `/26` | `10.10.99.62` | Point d'accès Wi-Fi 6 (`CORP-WIFI` / Visiteurs) |
| **`99`** | `PC Administrateur` | Poste admin | `10.10.99.61` | `/26` | `10.10.99.62` | Unique poste autorisé pour les flux d'administration |
| **`100`** | `SRV-AD 1` | VM (Cluster 1) | `10.10.100.1` | `/28` | `10.10.100.14` | Contrôleur de domaine principal (`AD DS` / `DNS`) |
| **`100`** | `SRV-COMPTA` | VM (Cluster 1) | `10.10.100.2` | `/28` | `10.10.100.14` | Serveur de comptabilité (`Sage 100c`) |
| **`100`** | `SRV-AD 2` | VM (Cluster 2) | `10.10.100.3` | `/28` | `10.10.100.14` | Contrôleur secondaire (`AD DS` / `DNS` / `RADIUS NPS`) |
| **`100`** | `SRV-APP` | VM (Cluster 2) | `10.10.100.4` | `/28` | `10.10.100.14` | Base de données métier (`SQL Server 2022`) |
| **`100`** | `SRV-VEEAM` | VM (Cluster 2) | `10.10.100.5` | `/28` | `10.10.100.14` | Serveur d'orchestration `Veeam Backup & Replication` |
| **`220`** | `Synology RackStation RS2423+` | NAS physique | `10.10.220.1` | `/29` | `10.10.220.6` | Dépôt local de sauvegardes (`4×4 To` RAID 10) |
| **`230`** | `SRV-SUPERVISION` | VM (Cluster 1) | `10.10.230.1` | `/29` | `10.10.230.6` | Monitoring `Centreon 24.10 LTS` (`Ubuntu 24.04 LTS`) |
| **`240`** | `SRV-HyperV 1` (NIC Cluster) | Hôte physique | `10.10.240.1` | `/30` | — | Lien dédié Cluster 1 (Heartbeat / Live Migration) |
| **`240`** | `SRV-HyperV 2` (NIC Cluster) | Hôte physique | `10.10.240.2` | `/30` | — | Lien dédié Cluster 1 (Heartbeat / Live Migration) |
| **`245`** | `SRV-HyperV 3` (NIC Cluster) | Hôte physique | `10.10.245.1` | `/30` | — | Lien dédié Cluster 2 (Heartbeat / Live Migration) |
| **`245`** | `SRV-HyperV 4` (NIC Cluster) | Hôte physique | `10.10.245.2` | `/30` | — | Lien dédié Cluster 2 (Heartbeat / Live Migration) |
