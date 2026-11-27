# 03 — Matrice de Filtrage & Politique ACL (Zero-Trust)

## Principes de filtrage périmétrique et inter-VLAN

Le routage inter-VLAN et le filtrage périmétrique sont assurés par le cluster haute disponibilité **FortiGate 60F (FGCP)** selon une approche **Zero-Trust / Deny-by-Default** :

- **Refus par défaut avec journalisation** (`Règle 999`) : tout flux non explicitement autorisé est bloqué (`DROP + LOG`).
- **Moindre privilège par département** : chaque service accède uniquement aux serveurs nécessaires à son activité.
- **Isolation des zones sensibles** : le `VLAN 220` (Backup) n'est joignable que par `SRV-VEEAM`, `Centreon` et le poste d'administration ; les interfaces de gestion (`VLAN 99`) sont restreintes à l'unique IP `10.10.99.61`.
- **Cloisonnement des visiteurs** : le `VLAN 210` dispose uniquement d'un accès Internet avec blocage explicite des plages privées (`RFC 1918`).

---

## Matrice complète des règles ACL

| ID | Nom / Flux logique | VLAN Source | IP / Réseau Source | VLAN Destination | IP / Réseau Destination | Protocole / Port | Action | Explication |
| :---: | :--- | :--- | :--- | :--- | :--- | :--- | :---: | :--- |
| **`1`** | Accès Services Socles (`AD / DNS`) | `VLAN 10, 20, 30, 40, 50, 99` | `192.168.10.0/28`<br>`192.168.20.0/28`<br>`192.168.30.0/28`<br>`192.168.40.0/28`<br>`192.168.50.0/28`<br>`10.10.99.0/26` | `VLAN 100` (`AD1 & AD2`) | `10.10.100.1`<br>`10.10.100.3` | `TCP 135` (RPC)<br>`TCP/UDP 53` (DNS)<br>`TCP 88` (Kerberos)<br>`TCP 389/636` (LDAP/S)<br>`TCP 445` (SMB GPO)<br>`UDP 123` (NTP) | **AUTORISER** | Autorise les postes et l'administration à s'authentifier et résoudre les noms via AD / DNS. |
| **`2`** | Accès Progiciel Comptabilité (`Sage 100`) | `VLAN 10` (Direction)<br>`VLAN 20` (RH & Compta) | `192.168.10.0/28`<br>`192.168.20.0/28` | `VLAN 100` (`SRV-COMPTA`) | `10.10.100.2` | `TCP 1433` (Ports applicatifs Sage 100) | **AUTORISER** | Restreint l'accès au serveur comptable aux seuls services habilités. |
| **`3`** | Accès Base de Données & App Métier (`SQL Server`) | `VLAN 10` (Direction)<br>`VLAN 30` (Commercial)<br>`VLAN 40` (Data)<br>`VLAN 50` (R&D) | `192.168.10.0/28`<br>`192.168.30.0/28`<br>`192.168.40.0/28`<br>`192.168.50.0/28` | `VLAN 100` (`SRV-APP`) | `10.10.100.4` | `TCP 443` (HTTPS)<br>`TCP 1433` (MS SQL) | **AUTORISER** | Autorise les départements métier concernés à interroger `SRV-APP` (RH & Compta exclus). |
| **`4`** | Sortie Internet & Cloud (`M365 / SharePoint / Exchange`) | `VLAN 10, 20, 30, 40, 50` | `192.168.10.0/28`<br>`192.168.20.0/28`<br>`192.168.30.0/28`<br>`192.168.40.0/28`<br>`192.168.50.0/28` | `WAN` (Internet) | `Any` (Internet) | `TCP 80` (HTTP)<br>`TCP/UDP 443` (HTTPS) | **AUTORISER** | Autorise la navigation Web et l'accès aux services SaaS Microsoft 365 pour tous les départements. |
| **`5`** | Isolation Wi-Fi Visiteurs / Nomades | `VLAN 210` (`VLAN-WIFI`) | `192.168.210.0/26` | `WAN` (Internet) | `Any` (`RFC 1918` bloqué) | `TCP/UDP 53` (DNS externe)<br>`TCP 80` (HTTP)<br>`TCP 443` (HTTPS)<br>`UDP 500/4500` (IPsec VPN) | **AUTORISER** | Autorise l'accès Internet visiteur sans aucun accès au réseau interne de l'entreprise. |
| **`6`** | Collecte des Sauvegardes (`Veeam -> Infra`) | `VLAN 100` (`SRV-VEEAM`) | `10.10.100.5` | `VLAN 100` (`SRV`)<br>`VLAN 99` (`Hyper-V 1 à 4`)<br>`VLAN 230` | `10.10.100.0/28`<br>`10.10.99.1` – `10.10.99.4`<br>`10.10.230.1` | `TCP 6160, 6162, 2500-3300` (Veeam Agents / Data Movers) | **AUTORISER** | Autorise la collecte des sauvegardes sur les hyperviseurs et VMs cibles avant transfert vers le NAS. |
| **`7`** | Stockage des Sauvegardes Locales (`Veeam -> NAS`) | `VLAN 100` (`SRV-VEEAM`) | `10.10.100.5` | `VLAN 220` (`Backup`) | `10.10.220.1` | `TCP 445` (SMB)<br>`TCP 3260` (iSCSI)<br>`TCP 6160, 2500-3300` (Veeam) | **AUTORISER** | Autorise l'écriture des sauvegardes locales depuis `SRV-VEEAM` vers le NAS Synology. |
| **`8`** | Externalisation Cloud Backup (`AWS S3/Glacier & Veeam Cloud Connect`) | `VLAN 100` (`SRV-VEEAM`) & `VLAN 220` (`Synology`) | `10.10.100.5`<br>`10.10.220.1` | `WAN` (`AWS, Veeam Cloud`) | Endpoints `AWS S3 / Glacier / Veeam Cloud` | `TCP 443` (HTTPS) | **AUTORISER** | Autorise la réplication chiffrée hors site vers AWS S3/Glacier et Veeam Cloud Connect. |
| **`9`** | Supervision de l'Infrastructure (`Centreon`) | `VLAN 230` (`SRV-SUPERVISION`) | `10.10.230.1` | `VLAN 99` (`Admin`)<br>`VLAN 100` (`SRV`)<br>`VLAN 220` (`Backup`) | `10.10.99.0/26`<br>`10.10.100.0/28`<br>`10.10.220.1` | `UDP 161` (SNMPv3)<br>`ICMP` (Ping)<br>`TCP 5666/5556` (NRPE / Agent) | **AUTORISER** | Autorise Centreon à collecter les métriques et états de santé sur les équipements supervisés. |
| **`10`** | Flux Inter-Nœuds Clusters Hyper-V | `VLAN 240 & VLAN 245` (`Clusters 1 & 2`) | `10.10.240.0/30`<br>`10.10.245.0/30` | `VLAN 240 & VLAN 245` | `10.10.240.0/30`<br>`10.10.245.0/30` | `TCP/UDP 3343` (Cluster)<br>`TCP 6600` (Live Migration)<br>`TCP 445` (SMB)<br>`ICMP` (Ping) | **AUTORISER** | Autorise les échanges inter-nœuds dédiés (règle appliquée localement sur les OS hôtes Hyper-V). |
| **`11`** | Administration Systèmes & Réseaux | `VLAN 99` (`PC Administrateur`) | `10.10.99.61` | `VLAN 99, 100, 220, 230` | `10.10.99.0/26`<br>`10.10.100.0/28`<br>`10.10.220.0/29`<br>`10.10.230.0/29` | `TCP 22` (SSH)<br>`TCP/UDP 3389` (RDP)<br>`TCP 443` (HTTPS DSM/FW/Centreon) | **AUTORISER** | Autorise uniquement le poste d'administration à gérer les équipements, hyperviseurs, NAS et supervision. |
| **`12`** | Update Server & Time Sync (`NTP`) | `VLAN 99, 100, 220, 230` | `10.10.99.0/26`<br>`10.10.100.0/28`<br>`10.10.220.0/29`<br>`10.10.230.0/29` | `WAN` (Internet) | Serveurs MAJ (`Windows Update`, `Ubuntu`, `Synology`) | `TCP/UDP 53` (DNS)<br>`TCP 80` (HTTP)<br>`TCP 443` (HTTPS)<br>`UDP 123` (NTP) | **AUTORISER** | Autorise les mises à jour de sécurité des serveurs/équipements ainsi que la synchronisation horaire. |
| **`13`** | Accès RADIUS Authentification | `VLAN 99` (`Aruba AP-505`) | `10.10.99.11`<br>`10.10.99.12` | `VLAN 100` (`SRV-AD 2`) | `10.10.100.3` | `UDP 1812 & 1813` (RADIUS) | **AUTORISER** | Autorise les bornes Wi-Fi HPE Aruba à authentifier les collaborateurs auprès du serveur RADIUS/NPS sur l'AD. |
| **`999`** | **Deny All** | `Any` | `0.0.0.0/0` | `Any` | `0.0.0.0/0` | `Any` | **BLOQUER (`DROP + LOG`)** | Blocage et journalisation de tout flux non explicitement autorisé. |

---

> [!IMPORTANT]
> ### Note d'architecture — Flux entrants télémétrie sondes maritimes
>
> L'activité cœur de DATASC.SEA repose sur la transmission temps réel de données issues de sondes maritimes vers l'infrastructure applicative (`SRV-APP`, `10.10.100.4` sous `Microsoft SQL Server 2022`).
>
> **Refus strict de l'exposition directe du port SQL (`TCP 1433`) sur Internet :**
> Aucune règle NAT / *port forwarding* `WAN -> TCP 1433` n'est autorisée dans cette matrice ACL. Exposer directement le moteur SQL Server sur le WAN créerait une surface d'attaque critique (attaques par force brute, exploitation de vulnérabilités protocolaires, injection directe dans le `VLAN 100`).
>
> **Architectures sécurisées retenues pour la remontée des flux terrain :**
> 1. **Option A — Tunnel VPN IPsec site-à-site ou passerelle embarquée (recommandée pour stations fixes / relais côtiers) :**
>    Les concentrateurs de sondes établissent un tunnel **VPN IPsec (IKEv2 / AES-256)** terminé sur le cluster **FortiGate 60F**. Le trafic déchiffré est ensuite filtré par une ACL dédiée n'autorisant que l'IP source du tunnel vers `10.10.100.4`.
> 2. **Option B — Passerelle API HTTPS (`TCP 443`) avec filtrage par IP source (adaptée aux sondes mobiles / cellulaires / satellitaires) :**
>    Les sondes poussent leurs payloads JSON/Protobuf vers une API d'ingestion exposée en **`HTTPS` (`TCP 443`)** derrière un reverse-proxy / WAF en zone dédiée, avec **liste blanche d'IP sources** (opérateurs/passerelles autorisés), authentification forte (certificats clients **mTLS** ou jetons signés) et inspection IPS FortiGate avant insertion contrôlée dans SQL Server.
