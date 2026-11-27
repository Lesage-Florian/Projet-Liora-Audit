# 03 — Matrice de Filtrage & Politique ACL (Zero-Trust)

## Principes de filtrage périmétrique et inter-VLAN

Le routage inter-VLAN et le filtrage périmétrique sont assurés par le cluster haute disponibilité **FortiGate 60F (FGCP)** selon une approche **Zero-Trust / Deny-by-Default** :

- **Refus par défaut avec journalisation** (`Règle 999`) : tout flux non explicitement autorisé est bloqué (`DROP + LOG`).
- **Moindre privilège par département** : chaque service accède uniquement aux serveurs nécessaires à son activité.
- **Isolation des zones sensibles** : le `VLAN 220` (Backup) n'est joignable que par `SRV-VEEAM`, `Centreon` et le poste d'administration ; les interfaces de gestion (`VLAN 99`) sont restreintes à l'unique IP `10.10.99.61`.
- **Cloisonnement des visiteurs** : le `VLAN 210` dispose uniquement d'un accès Internet avec blocage explicite des plages privées (`RFC 1918`).

---

## Matrice complète des règles ACL

| ID | Flux logique & Description | Source (VLAN / IP) | Destination (VLAN / IP) | Ports & Action |
| :---: | :--- | :--- | :--- | :--- |
| **`1`** | **Services Socles (`AD / DNS`)**<br>Authentification et résolution DNS pour tous les postes et l'admin. | `VLAN 10, 20, 30, 40, 50` (`192.168.x.0/28`)<br>`VLAN 99` (`10.10.99.0/26`) | `VLAN 100` (`AD1 & AD2`)<br>`10.10.100.1`, `10.10.100.3` | **AUTORISER**<br>`TCP 135` (RPC), `TCP/UDP 53` (DNS), `TCP 88` (Kerberos), `TCP 389/636` (LDAP/S), `TCP 445` (SMB GPO), `UDP 123` (NTP) |
| **`2`** | **Progiciel Comptabilité (`Sage 100`)**<br>Accès restreint aux seuls départements Direction et RH/Compta. | `VLAN 10` (`192.168.10.0/28`)<br>`VLAN 20` (`192.168.20.0/28`) | `VLAN 100` (`SRV-COMPTA`)<br>`10.10.100.2` | **AUTORISER**<br>`TCP 1433` (Sage 100) |
| **`3`** | **Base de Données & App Métier (`SQL Server`)**<br>Accès pour Direction, Commercial, Data et R&D (RH/Compta exclus). | `VLAN 10, 30, 40, 50`<br>`192.168.{10,30,40,50}.0/28` | `VLAN 100` (`SRV-APP`)<br>`10.10.100.4` | **AUTORISER**<br>`TCP 443` (HTTPS)<br>`TCP 1433` (MS SQL) |
| **`4`** | **Sortie Internet & Cloud (`M365`)**<br>Navigation Web et accès SaaS Microsoft 365 pour les utilisateurs. | `VLAN 10, 20, 30, 40, 50`<br>`192.168.{10..50}.0/28` | `WAN` (Internet)<br>`Any` | **AUTORISER**<br>`TCP 80` (HTTP)<br>`TCP/UDP 443` (HTTPS) |
| **`5`** | **Isolation Wi-Fi Visiteurs / Nomades**<br>Accès Internet seul avec blocage strict du réseau interne (`RFC 1918`). | `VLAN 210` (`VLAN-WIFI`)<br>`192.168.210.0/26` | `WAN` (Internet)<br>`Any` (`RFC 1918` bloqué) | **AUTORISER**<br>`TCP/UDP 53` (DNS), `TCP 80/443`, `UDP 500/4500` (IPsec VPN) |
| **`6`** | **Collecte des Sauvegardes (`Veeam -> Infra`)**<br>Collecte des sauvegardes sur les hyperviseurs et VMs cibles. | `VLAN 100` (`SRV-VEEAM`)<br>`10.10.100.5` | `VLAN 100` (`10.10.100.0/28`)<br>`VLAN 99` (`10.10.99.1` – `.4`)<br>`VLAN 230` (`10.10.230.1`) | **AUTORISER**<br>`TCP 6160, 6162, 2500-3300` (Veeam Agents/Data Movers) |
| **`7`** | **Stockage Sauvegardes Locales (`Veeam -> NAS`)**<br>Écriture des dépôts locaux vers le NAS Synology isolé. | `VLAN 100` (`SRV-VEEAM`)<br>`10.10.100.5` | `VLAN 220` (`Backup`)<br>`10.10.220.1` | **AUTORISER**<br>`TCP 445` (SMB), `TCP 3260` (iSCSI), `TCP 6160, 2500-3300` |
| **`8`** | **Externalisation Cloud Backup**<br>Réplication chiffrée hors site vers AWS S3/Glacier et Veeam Cloud Connect. | `VLAN 100` (`10.10.100.5`)<br>`VLAN 220` (`10.10.220.1`) | `WAN` (Cloud)<br>Endpoints `AWS S3 / Glacier / Veeam Cloud` | **AUTORISER**<br>`TCP 443` (HTTPS) |
| **`9`** | **Supervision Infrastructure (`Centreon`)**<br>Collecte SNMPv3, Ping et agents sur l'infrastructure cible. | `VLAN 230` (`SRV-SUPERVISION`)<br>`10.10.230.1` | `VLAN 99` (`10.10.99.0/26`)<br>`VLAN 100` (`10.10.100.0/28`)<br>`VLAN 220` (`10.10.220.1`) | **AUTORISER**<br>`UDP 161` (SNMPv3), `ICMP` (Ping), `TCP 5666/5556` (NRPE) |
| **`10`** | **Flux Inter-Nœuds Clusters Hyper-V**<br>Heartbeat et Live Migration (règle locale sur les OS hôtes Hyper-V). | `VLAN 240` (`10.10.240.0/30`)<br>`VLAN 245` (`10.10.245.0/30`) | `VLAN 240` (`10.10.240.0/30`)<br>`VLAN 245` (`10.10.245.0/30`) | **AUTORISER**<br>`TCP/UDP 3343` (Cluster), `TCP 6600` (Live Migration), `TCP 445`, `ICMP` |
| **`11`** | **Administration Systèmes & Réseaux**<br>Accès de gestion restreint exclusivement au poste d'administration. | `VLAN 99` (`PC Admin`)<br>`10.10.99.61` | `VLAN 99, 100, 220, 230`<br>(`10.10.{99,100,220,230}.0`) | **AUTORISER**<br>`TCP 22` (SSH), `TCP/UDP 3389` (RDP), `TCP 443` (HTTPS) |
| **`12`** | **Update Server & Time Sync (`NTP`)**<br>Mises à jour de sécurité et synchronisation horaire des serveurs. | `VLAN 99, 100, 220, 230` | `WAN` (Internet)<br>Serveurs MAJ (`Windows Update`, `Ubuntu`, `Synology`) | **AUTORISER**<br>`TCP/UDP 53` (DNS), `TCP 80/443`, `UDP 123` (NTP) |
| **`13`** | **Accès RADIUS Authentification**<br>Authentification Wi-Fi `802.1X` des bornes Aruba auprès de l'AD (`NPS`). | `VLAN 99` (`Aruba AP-505`)<br>`10.10.99.11`, `10.10.99.12` | `VLAN 100` (`SRV-AD 2`)<br>`10.10.100.3` | **AUTORISER**<br>`UDP 1812 & 1813` (RADIUS) |
| **`999`** | **Deny All**<br>Blocage et journalisation de tout flux non explicitement autorisé. | `Any` (`0.0.0.0/0`) | `Any` (`0.0.0.0/0`) | **BLOQUER (`DROP + LOG`)**<br>`Any` |

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
