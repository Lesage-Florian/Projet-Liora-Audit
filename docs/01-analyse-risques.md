# 01 — Analyse des Risques du Système d'Information

## Synthèse de l'audit

L'audit initial du système d'information de **DATASC.SEA** (Bayonne) porte sur **15 domaines** (localisation, locaux, sauvegardes, NAS, infrastructure, adressage, box Internet, serveurs, machines virtuelles, antivirus, serveur de fichiers, Active Directory, mots de passe, sensibilisation, messagerie). Il recense **41 risques** au total.

| Indicateur | Valeur relevée |
| :--- | :--- |
| **Total des risques identifiés** | **41** |
| **Risques critiques (`Score >= 9/10`)** | **28** (dont **16** au niveau maximal `10/10` et **12** à `9/10`) |
| **Risques élevés (`7` à `8/10`)** | **11** (4 à `8/10`, 7 à `7/10`) |
| **Risques moyens (`<= 6/10`)** | **2** (`6/10`) |
| **Impact moyen** | **4,44 / 5** |
| **Probabilité moyenne** | **4,37 / 5** |

> **Méthode de scoring :** $\text{Score (/10)} = \text{Impact (/5)} + \text{Probabilité (/5)}$.

---

## Les 10 priorités consolidées

Pour structurer le plan de remédiation, les 28 risques critiques ont été regroupés par équipement et thématique en **10 chantiers prioritaires** :

1. **Sauvegardes inutilisables** *(ID 3, 4)* — Disques saturés et rétention limitée à 7 jours, rendant toute reprise impossible.
2. **Hyperviseur HV1** *(ID 18, 19, 20)* — Obsolescence matérielle, OS et applicative menaçant l'ensemble des VMs hébergées.
3. **NAS de sauvegarde physique** *(ID 15, 16, 17)* — Obsolescence du matériel, du système (DSM 2015) et de la solution de backup.
4. **Serveur de fichiers & gouvernance des données** *(ID 9, 10, 11, 32)* — Obsolescence complète et partages ouverts à « Tout le monde » (non-conformité RGPD).
5. **Contrôleurs de domaine & Active Directory** *(ID 12, 13, 14, 21, 22, 34)* — Obsolescence matérielle/logicielle, absence d'OU, de convention de nommage et de cloisonnement des droits.
6. **Segmentation et cloisonnement réseau** *(ID 7)* — Réseau plat permettant à n'importe quel poste d'accéder aux flux sensibles (Direction, RH, Compta).
7. **Sécurité de la messagerie & Phishing** *(ID 41)* — Absence de messagerie d'entreprise dédiée (usage d'adresses `orange.fr`) exposant à l'usurpation d'identité et à la fraude au virement.
8. **Serveur de comptabilité** *(ID 23, 24)* — Obsolescence conjointe de l'OS et de Sage 100.
9. **Serveurs applicatif et web** *(ID 25, 26, 27, 28)* — Obsolescence des systèmes et applicatifs sur `SRV-APP` et `SRV-WEB`.
10. **Supervision du SI** *(ID 29, 30)* — Obsolescence de l'OS et de l'outil de monitoring, réduisant la réactivité en cas d'incident.

> [!NOTE]
> Les risques physiques **Inondation** (*ID 1*, score `6/10`) et **Intrusion / dégradation physique** (*ID 2*, score `7/10`) présentent un impact maximal (`5/5`) et restent placés sous surveillance active. Ils sont adressés par la création d'une salle serveur dédiée et l'externalisation des sauvegardes hors site.

---

## Vue synthétique des 41 risques

| ID | Vulnérabilité | Impact / Prob. | Score | Criticité |
| :---: | :--- | :---: | :---: | :---: |
| **1** | Inondation (crue de l'Adour, Bayonne) | `5` / `1` | **6** | Moyen |
| **2** | Intrusion / dégradation physique (baie ouverte) | `5` / `2` | **7** | Élevé |
| **3** | Disques durs pleins (sauvegardes bloquées) | `5` / `5` | **10** | **Critique** |
| **4** | Rétention trop courte (`7 jours`) | `5` / `5` | **10** | **Critique** |
| **5** | Hardware limité (équipement grand public) | `3` / `5` | **8** | Élevé |
| **6** | Panne d'un équipement (absence de redondance) | `4` / `4` | **8** | Élevé |
| **7** | Absence de VLAN (réseau plat) | `5` / `4` | **9** | **Critique** |
| **8** | Pas de relais Internet (box grand public) | `4` / `3` | **7** | Élevé |
| **9** | `SRV-FICHIERS` — Obsolescence (matériel) | `5` / `5` | **10** | **Critique** |
| **10** | `SRV-FICHIERS` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **11** | `SRV-FICHIERS` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **12** | `SRV-AD2` — Obsolescence (matériel) | `5` / `5` | **10** | **Critique** |
| **13** | `SRV-AD2` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **14** | `SRV-AD2` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **15** | `NAS-SAUVEGARDE` — Obsolescence (matériel) | `5` / `5` | **10** | **Critique** |
| **16** | `NAS-SAUVEGARDE` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **17** | `NAS-SAUVEGARDE` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **18** | `HV1` — Obsolescence (matériel) | `5` / `5` | **10** | **Critique** |
| **19** | `HV1` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **20** | `HV1` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **21** | `SRV-AD` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **22** | `SRV-AD` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **23** | `SRV-COMPTA` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **24** | `SRV-COMPTA` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **25** | `SRV-APP` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **26** | `SRV-APP` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **27** | `SRV-WEB` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **28** | `SRV-WEB` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **29** | `SRV-SUPERVISION` — Obsolescence (système) | `5` / `5` | **10** | **Critique** |
| **30** | `SRV-SUPERVISION` — Obsolescence (application) | `4` / `5` | **9** | **Critique** |
| **31** | Antivirus grand public non maintenu | `5` / `3` | **8** | Élevé |
| **32** | Serveur de fichiers ouvert à tous | `5` / `4` | **9** | **Critique** |
| **33** | Comptes orphelins dans l'Active Directory | `4` / `3` | **7** | Élevé |
| **34** | Organisation de l'Active Directory | `5` / `5` | **10** | **Critique** |
| **35** | Stratégie de mots de passe faible | `4` / `4` | **8** | Élevé |
| **36** | Absence de suivi des équipements partagés | `2` / `4` | **6** | Moyen |
| **37** | Absence de charte informatique & télétravail | `3` / `4` | **7** | Élevé |
| **38** | Circulation des données sans protection (USB) | `4` / `3` | **7** | Élevé |
| **39** | Accès à distance non sécurisé | `5` / `2` | **7** | Élevé |
| **40** | Accès Wi-Fi non sécurisé (WEP / post-it) | `4` / `3` | **7** | Élevé |
| **41** | Mails de phishing / usurpation d'identité | `4` / `5` | **9** | **Critique** |

---

## Détail des scénarios, impacts et solutions retenues

### 1. Locaux & Infrastructure physique (`ID 1`, `2`, `5`, `6`)

- **ID 1 — Inondation** *(Impact `5/5`, Prob. `1/5`, Score **`6/10` — Moyen**)*
  - **Scénario :** Lors de fortes pluies, **l'Adour** sort de son lit et inonde les locaux de l'entreprise (Bayonne).
  - **Effets & Cause :** Destruction des locaux et du SI, perte potentielle des données et du matériel ; locaux en rez-de-chaussée en zone inondable.
  - **Solution :** Architecture hybride avec réplication hors site (`AWS S3 / Glacier` & `Veeam Cloud Connect`) et surélévation en baie.
- **ID 2 — Intrusion / dégradation physique** *(Impact `5/5`, Prob. `2/5`, Score **`7/10` — Élevé**)*
  - **Scénario :** Vol de matériel/données ou accident (un employé renverse son café sur la baie placée dans l'open space).
  - **Effets & Cause :** Exfiltration de données sensibles, vol ou destruction du matériel ; locaux non cloisonnés, baie accessible à tous.
  - **Solution :** Transformation de la salle de réunion 1 en salle serveur fermée avec baie `42U` verrouillable.
- **ID 5 — Hardware limité** *(Impact `3/5`, Prob. `5/5`, Score **`8/10` — Élevé**)*
  - **Scénario :** Matériel inadapté à la croissance de l'entreprise.
  - **Effets & Cause :** Ralentissements et impossibilité d'accompagner la montée en charge ; équipement grand public.
  - **Solution :** Renouvellement par une infrastructure scalable (`4× Dell PowerEdge R350`).
- **ID 6 — Panne d'un équipement** *(Impact `4/5`, Prob. `4/5`, Score **`8/10` — Élevé**)*
  - **Scénario :** Un ou plusieurs équipements critiques tombent en panne.
  - **Effets & Cause :** Arrêt de la production et blocage de l'activité ; aucune redondance serveur, passerelle unique Livebox grand public.
  - **Solution :** Redondance complète (clusters Hyper-V, FGCP, LACP 20G, double onduleur) et support constructeur dédié.

### 2. Sauvegardes (`ID 3`, `4`)

- **ID 3 — Disques durs pleins** *(Impact `5/5`, Prob. `5/5`, Score **`10/10` — Critique**)*
  - **Scénario :** Espace disque insuffisant pour assurer la continuité des sauvegardes.
  - **Effets & Cause :** Perte de données, absence de sauvegarde de la production récente ; disque dur saturé et gestion défaillante.
  - **Solution :** Déploiement d'un Synology `RS2423+` (`4×4 To` RAID 10) et stratégie `3-2-1-1-0` (VM → NAS → Cloud).
- **ID 4 — Rétention trop courte** *(Impact `5/5`, Prob. `5/5`, Score **`10/10` — Critique**)*
  - **Scénario :** La restauration des données de la semaine précédente est impossible (`7 jours` de rétention seulement).
  - **Effets & Cause :** Perte définitive de données en cas d'incident détecté tardivement ; rétention insuffisante sur le NAS existant.
  - **Solution :** Rétention étagée : `24h` (snapshots), `14j` quotidienne, `30j` immuable S3, `3 ans` / `10 ans` (compta) Glacier.

### 3. Réseau & Connectivité (`ID 7`, `8`, `39`, `40`)

- **ID 7 — Absence de VLAN** *(Impact `5/5`, Prob. `4/5`, Score **`9/10` — Critique**)*
  - **Scénario :** Un employé mécontent consulte et partage les fiches de paie de l'intégralité des services.
  - **Effets & Cause :** Fuite ou lecture de données sensibles (Direction, RH, Compta) ; réseau plat `192.168.1.0/24` sans segmentation ni filtrage.
  - **Solution :** Segmentation en **12 VLANs** avec filtrage inter-VLAN par ACL sur FortiGate (`Deny All` par défaut).
- **ID 8 — Pas de relais Internet** *(Impact `4/5`, Prob. `3/5`, Score **`7/10` — Élevé**)*
  - **Scénario :** Coupure d'accès Internet avec un support grand public pouvant mettre jusqu'à 2 semaines.
  - **Effets & Cause :** Perte totale d'accès Internet et rupture de transmission des données ; box grand public sans garantie de rétablissement.
  - **Solution :** Liaison pro **FTTO** (`1 Gbit/s`, GTR 4h) + **FTTH** de secours sur routeurs distincts avec failover automatique.
- **ID 39 — Accès à distance non sécurisé** *(Impact `5/5`, Prob. `2/5`, Score **`7/10` — Élevé**)*
  - **Scénario :** Un employé se connecte au SI depuis un Wi-Fi public sans tunnel chiffré.
  - **Effets & Cause :** Interception de données en clair et accès tiers malveillant ; absence de VPN encadré (usage de TeamViewer).
  - **Solution :** Remplacement par **VPN IPsec / SSL** terminé sur FortiGate (`FortiClient`).
- **ID 40 — Accès Wi-Fi non sécurisé** *(Impact `4/5`, Prob. `3/5`, Score **`7/10` — Élevé**)*
  - **Scénario :** Un visiteur/livreur photographie le code Wi-Fi et accède au LAN interne.
  - **Effets & Cause :** Intrusion directe dans le réseau interne ; Wi-Fi WEP avec clé partagée affichée, absence de GPO Wi-Fi.
  - **Solution :** Bornes **HPE Aruba AP-505** en **WPA3-Enterprise (`802.1X` / RADIUS)** + SSID visiteurs isolé (`VLAN 210`).

### 4. Obsolescence Serveurs, Hyperviseur & Applicatifs (`ID 9` à `30`)

Les 22 risques d'obsolescence concernent l'impossibilité de maintenir à jour les équipements, systèmes d'exploitation et logiciels métier, ainsi que l'absence de support matériel en `J+1` :

- **ID 9, 10, 11 — `SRV-FICHIERS` (Matériel `10/10`, OS `10/10`, App `9/10`)** : Remplacement du matériel avec support constructeur et migration des partages vers **SharePoint Online / OneDrive (Microsoft 365)**.
- **ID 12, 13, 14 — `SRV-AD2` (Matériel `10/10`, OS `10/10`, App `9/10`)** : Remplacement matériel (`Dell R350`) avec Dell ProSupport et déploiement sous `Windows Server 2025`.
- **ID 15, 16, 17 — `NAS-SAUVEGARDE` (Matériel `10/10`, OS `10/10`, App `9/10`)** : Remplacement du `DS214Play` (DSM 2015) par un **Synology RS2423+** supporté + **Veeam Data Platform Premium**.
- **ID 18, 19, 20 — `HV1` (Matériel `10/10`, OS `10/10`, App `9/10`)** : Remplacement de l'hôte unique `PowerEdge R440` par **4× Dell PowerEdge R350** (Dell ProSupport) en **2 clusters Hyper-V HA** sous `Windows Server 2025`.
- **ID 21, 22 — `SRV-AD` (OS `10/10`, App `9/10`)** : Reconstruction des contrôleurs de domaine sur `Windows Server 2025`.
- **ID 23, 24 — `SRV-COMPTA` (OS `10/10`, App `9/10`)** : Migration vers `Windows Server 2025` et déploiement de `Sage 100c`.
- **ID 25, 26 — `SRV-APP` (OS `10/10`, App `9/10`)** : Migration vers `Windows Server 2025` et `Microsoft SQL Server 2022`.
- **ID 27, 28 — `SRV-WEB` (OS `10/10`, App `9/10`)** : Externalisation du site Web vers **OVHcloud Pro** (plateforme maintenue et à jour).
- **ID 29, 30 — `SRV-SUPERVISION` (OS `10/10`, App `9/10`)** : Migration vers **Centreon 24.10 LTS** sur `Ubuntu Server 24.04 LTS`.

### 5. Identités, Postes, Messagerie & Gouvernance (`ID 31` à `38`, `41`)

- **ID 31 — Antivirus grand public** *(Impact `5/5`, Prob. `3/5`, Score **`8/10` — Élevé**)*
  - **Scénario & Effets :** Un collaborateur RH ouvre une pièce jointe malveillante ; intrusion d'un **ransomware** et chiffrement des données (antivirus non maintenu).
  - **Solution :** Déploiement d'un EDR professionnel (**Microsoft Defender for Business**, inclus dans M365 Business Premium).
- **ID 32 — Serveur de fichiers ouvert à tous** *(Impact `5/5`, Prob. `4/5`, Score **`9/10` — Critique**)*
  - **Scénario & Effets :** Suppression d'un dossier critique ou consultation de données RH ; perte de confidentialité et non-conformité **RGPD**.
  - **Solution :** Migration vers **SharePoint Online / OneDrive** avec cloisonnement par service, rôles et moindre privilège.
- **ID 33 — Comptes orphelins** *(Impact `4/5`, Prob. `3/5`, Score **`7/10` — Élevé**)*
  - **Scénario & Effets :** Un ancien salarié se connecte encore avec son compte resté actif ; exfiltration de données.
  - **Solution :** Suppression des comptes orphelins et procédure systématique d'offboarding AD/M365.
- **ID 34 — Organisation de l'Active Directory** *(Impact `5/5`, Prob. `5/5`, Score **`10/10` — Critique**)*
  - **Scénario & Effets :** Un collaborateur possède des privilèges excessifs sur le domaine ; accès illégitime à des données sensibles (absence d'OU et de convention de nommage).
  - **Solution :** Refonte complète de l'AD en **Unités Organisationnelles (OU)** avec convention de nommage et moindre privilège.
- **ID 35 — Stratégie de mots de passe faible** *(Impact `4/5`, Prob. `4/5`, Score **`8/10` — Élevé**)*
  - **Scénario & Effets :** Compromission d'un compte par force brute en raison d'un mot de passe trop simple et de l'absence de MFA.
  - **Solution :** Stratégie de mots de passe conforme aux **recommandations ANSSI** + généralisation de la **MFA**.
- **ID 36 — Absence de suivi des équipements partagés** *(Impact `2/5`, Prob. `4/5`, Score **`6/10` — Moyen**)*
  - **Scénario & Effets :** Utilisation d'un PC partagé resté connecté sur la session précédente ; impossible d'attribuer les actions effectuées.
  - **Solution :** Comptes nominatifs temporaires et feuille de prêt pour les équipements partagés.
- **ID 37 — Absence de charte informatique & télétravail** *(Impact `3/5`, Prob. `4/5`, Score **`7/10` — Élevé**)*
  - **Scénario & Effets :** Après une attaque ransomware, l'assureur refuse la prise en charge (absence de cadre formel).
  - **Solution :** Déploiement des **chartes informatique et télétravail** + campagnes régulières de sensibilisation cybersécurité.
- **ID 38 — Circulation des données sans protection** *(Impact `4/5`, Prob. `3/5`, Score **`7/10` — Élevé**)*
  - **Scénario & Effets :** Transfert de fichiers pro/perso via une clé USB infectée ; infection par malware et fuite de données.
  - **Solution :** Blocage de l'utilisation et de l'écriture sur supports **USB via GPO**.
- **ID 41 — Mails de phishing** *(Impact `4/5`, Prob. `5/5`, Score **`9/10` — Critique**)*
  - **Scénario & Effets :** Usurpation d'identité du dirigeant (fraude au virement) ; fuite de données et préjudice financier (adresses `orange.fr`).
  - **Solution :** Migration vers **Exchange Online (Microsoft 365 Business Premium)** avec **nom de domaine propre OVH**, filtrage anti-spam/anti-phishing et procédure de vérification des paiements.
