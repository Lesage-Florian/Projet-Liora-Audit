# 04 — Stratégie de Sauvegarde & Dimensionnement Système

## La règle `3-2-1-1-0`

Pour éliminer les risques critiques de perte de données (*ID 3* : disques pleins, *ID 4* : rétention 7 jours) et couvrir le risque majeur d'inondation locale de l'Adour (*ID 1*), la refonte applique le standard **`3-2-1-1-0`** piloté par **Veeam Data Platform Premium** :

| Pilier | Implémentation chez DATASC.SEA |
| :--- | :--- |
| **3 copies** | Production (Clusters Hyper-V) + copie locale (`Synology RS2423+`) + double copie Cloud (`AWS S3` & `Veeam Cloud Connect`). |
| **2 supports différents** | Stockage disque local (`RAID 10`) et stockage objet / Cloud externalisé. |
| **1 copie hors site** | Réplication quotidienne et hebdomadaire vers `AWS S3 / Glacier` et `Veeam Cloud Connect`. |
| **1 copie immuable** | Verrouillage en écriture **Object Lock (WORM)** pendant **`30 jours`** sur `AWS S3` (ainsi que snapshots immuables sur le NAS) rendant le chiffrement par ransomware impossible. |
| **0 erreur** | Supervision des jobs + **test de restauration hebdomadaire** (fichier) et **mensuel** (serveur complet). |

---

## Inventaire et dimensionnement Hardware, VMs & SaaS

L'infrastructure est bâtie sur une **architecture hybride** : les charges critiques nécessitant faible latence et contrôle local restent sur site sur **2 clusters Hyper-V redondants** (`4× Dell PowerEdge R350`), tandis que la collaboration et la messagerie migrent vers le SaaS.

| Équipement / VM | Type | RAM | CPU / vCPU | Stockage | Système d'exploitation | Rôle / Applications |
| :--- | :--- | :--- | :--- | :--- | :--- | :--- |
| `SRV-HyperV (1)` — Dell R350 | Physique (Cluster 1) | `32 Go` | `16 vCPU` | `480 Go SSD` (OS) + `2 To SSD` (VM) | `Windows Server Standard 2025` | Hôte Hyper-V (Cluster 1) |
| `SRV-HyperV (2)` — Dell R350 | Physique (Cluster 1) | `32 Go` | `16 vCPU` | `480 Go SSD` (OS) + `2 To SSD` (VM) | `Windows Server Standard 2025` | Hôte Hyper-V (Cluster 1) |
| `SRV-HyperV (3)` — Dell R350 | Physique (Cluster 2) | `32 Go` | `16 vCPU` | `480 Go SSD` (OS) + `2 To SSD` (VM) | `Windows Server Standard 2025` | Hôte Hyper-V (Cluster 2) |
| `SRV-HyperV (4)` — Dell R350 | Physique (Cluster 2) | `32 Go` | `16 vCPU` | `480 Go SSD` (OS) + `2 To SSD` (VM) | `Windows Server Standard 2025` | Hôte Hyper-V (Cluster 2) |
| `SRV-AD 1` | VM (Cluster 1) | `4 Go` | `2 vCPU` | `80 Go SSD` | `Windows Server Standard 2025` | Contrôleur de domaine principal (`AD DS` / `DNS`) |
| `SRV-COMPTA` | VM (Cluster 1) | `12 Go` | `4 vCPU` | `250 Go SSD` | `Windows Server Standard 2025` | Comptabilité (`Sage 100c`) |
| `SRV-SUPERVISION` | VM (Cluster 1) | `8 Go` | `4 vCPU` | `120 Go SSD` | `Ubuntu Server 24.04 LTS` | Supervision SI (`Centreon 24.10 LTS`) |
| `SRV-AD 2` | VM (Cluster 2) | `4 Go` | `2 vCPU` | `80 Go SSD` | `Windows Server Standard 2025` | Contrôleur secondaire (`AD DS` / `DNS` / `RADIUS`) |
| `SRV-APP` | VM (Cluster 2) | `12 Go` | `4 vCPU` | `500 Go SSD` | `Windows Server Standard 2025` | Base de données métier (`SQL Server 2022`) |
| `SRV-VEEAM` | VM (Cluster 2) | `8 Go` | `4 vCPU` | `80 Go SSD` | `Windows Server Standard 2025` | Console & orchestration `Veeam Backup & Replication` |
| `Synology RS2423+` | NAS rackable (`VLAN 220`) | `8 Go` | `4 cœurs` | `4× 4 To HDD` en **RAID 10 (`8 To` utiles)** | `Synology DSM` | Dépôt local des sauvegardes & snapshots immuables |
| `SRV-WEB OVH` | Cloud (OVHcloud Pro) | Hébergé | — | — | `OVHcloud` | Hébergement du site Web vitrine |
| `SharePoint / OneDrive` | SaaS (`Microsoft 365`) | Cloud | — | — | `Microsoft 365 Business Premium` | Remplacement du serveur de fichiers local |
| `Exchange Online` | SaaS (`Microsoft 365`) | Cloud | — | — | `Microsoft 365 Business Premium` | Messagerie professionnelle (domaine OVH) |

---

## Planning des sauvegardes et politique de rétention

Les flux de sauvegarde transitent exclusivement via `SRV-VEEAM` (`10.10.100.5`) vers le `VLAN 220` isolé (`10.10.220.1`), inaccessibles depuis les postes utilisateurs :

| Serveur / Système | Snapshot Horaire | Copie Quotidienne (Synology) | Copie Quotidienne (`AWS S3`) | Copie Hebdomadaire (`AWS S3` + `Veeam Cloud`) | Copie Mensuelle (`AWS Glacier`) | Copie Annuelle (`AWS Glacier`) | Politique de rétention |
| :--- | :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **`SRV-AD 1`** | — | `23h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage long terme AWS : **`3 ans`**. |
| **`SRV-AD 2`** | — | `23h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage long terme AWS : **`3 ans`**. |
| **`SRV-COMPTA`** | `10h - 14h - 18h` | `22h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Horaire `24 h`, quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage légal AWS : **`10 ans`**. |
| **`SRV-APP`** | `10h - 14h - 18h` | `22h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Horaire `24 h`, quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage long terme AWS : **`3 ans`**. |
| **`SRV-SUPERVISION`** | — | `23h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage long terme AWS : **`3 ans`**. |
| **`SRV-VEEAM`** | — | `23h00` | `01h00` | Week-end | `1er du mois` | `1er janvier` | Quotidienne `14 j`, hebdomadaire immuable `30 j`, archivage long terme AWS : **`3 ans`**. |
| **Web (`OVHcloud`)** | Paramétrage OVH | — | — | — | — | — | Sauvegardes et snapshots automatisés selon politique `OVHcloud`. |
| **Fichiers & Mails (`M365`)** | Paramétrage M365 | `Toutes les 4 h` (Veeam) | — | Week-end (`Veeam Cloud Connect`) | — | — | Historique des versions M365, corbeille SharePoint `93 j` + sauvegarde externalisée via **Veeam Cloud Connect**. |

---

## Procédures de vérification et reprise après sinistre

- **Contrôle continu (0 erreur)** : chaque échec de tâche Veeam remonte une alerte vers `Centreon 24.10` et l'administrateur.
- **Tests de restauration planifiés** :
  - **Hebdomadaire** : restauration unitaire d'un fichier test pour valider l'intégrité des dépôts.
  - **Mensuel** : restauration complète d'une machine virtuelle isolée pour valider la capacité de reprise à froid.
- **Scénarios de reprise** :
  - *Panne locale ou erreur humaine* : restauration rapide à faible RTO depuis le **Synology RS2423+** (liens 10G/LACP 20G).
  - *Ransomware* : restauration saine garantie par les **copies immuables 30 jours** sur AWS S3 et les snapshots verrouillés du NAS.
  - *Sinistre majeur sur site (ex. crue de l'Adour)* : récupération intégrale des données et VMs depuis **AWS S3 / Glacier** ou **Veeam Cloud Connect**.
