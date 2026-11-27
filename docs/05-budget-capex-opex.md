# 05 — Budget CAPEX / OPEX & Projection TCO sur 3 ans

## Synthèse financière globale

Le chiffrage consolide l'ensemble des investissements matériels/licences (**CAPEX**) et des services récurrents (**OPEX**) nécessaires au déploiement et au maintien en condition opérationnelle de l'infrastructure cible.

| Composante | Type | Montant HT | Part Année 1 |
| :--- | :--- | :---: | :---: |
| **Investissements Matériels & Licences** | **CAPEX** (ponctuel) | **`40 844 €`** | `67,4 %` |
| **Services Récurrents & Abonnements** | **OPEX** (annuel) | **`19 718 € / an`** | `32,6 %` |
| **Budget Global Consolidé (Année 1)** | **CAPEX + OPEX** | **`60 562 €`** | `100 %` |
| **Coût Total de Possession (TCO 3 ans)** | **CAPEX + 3 × OPEX** | **`99 998 €`** | — |

### Projection TCO sur 3 ans

| Période | Investissement CAPEX HT | Coûts récurrents OPEX HT | Coût annuel HT | Cumul TCO HT |
| :--- | :---: | :---: | :---: | :---: |
| **Année 1** | `40 844 €` | `19 718 €` | `60 562 €` | `60 562 €` |
| **Année 2** | `0 €` | `19 718 €` | `19 718 €` | `80 280 €` |
| **Année 3** | `0 €` | `19 718 €` | `19 718 €` | **`99 998 €`** |

---

## Détail des dépenses d'investissement (CAPEX)

| Catégorie | Quantité | Équipement / Désignation | Prix unitaire HT | Montant Total HT |
| :--- | :---: | :--- | :---: | :---: |
| **Postes de travail** | 4 | Postes Direction *(parc conservé)* | `0 €` | `0 €` |
| **Postes de travail** | 4 | Postes RH & Compta *(parc conservé)* | `0 €` | `0 €` |
| **Postes de travail** | 4 | Postes Commercial *(parc conservé)* | `0 €` | `0 €` |
| **Postes de travail** | 4 | Postes Data Analyst *(parc conservé)* | `0 €` | `0 €` |
| **Postes de travail** | 4 | Postes Recherche & Développement *(parc conservé)* | `0 €` | `0 €` |
| **Postes de travail** | 1 | Poste Administrateur Systèmes & Réseaux | `1 500 €` | `1 500 €` |
| **Serveurs (Compute)** | 1 | `Dell PowerEdge R350` / 32 Go (`SRV-HyperV 1`) | `4 500 €` | `4 500 €` |
| **Serveurs (Compute)** | 1 | `Dell PowerEdge R350` / 32 Go (`SRV-HyperV 2`) | `4 500 €` | `4 500 €` |
| **Serveurs (Compute)** | 1 | `Dell PowerEdge R350` / 32 Go (`SRV-HyperV 3`) | `4 500 €` | `4 500 €` |
| **Serveurs (Compute)** | 1 | `Dell PowerEdge R350` / 32 Go (`SRV-HyperV 4`) | `4 500 €` | `4 500 €` |
| **Stockage Backup** | 1 | `Synology RackStation RS2423+` + 4× HDD 4 To | `5 000 €` | `5 000 €` |
| **Wi-Fi Professionnel** | 1 | Borne `HPE Aruba AP-505 (1)` | `1 017 €` | `1 017 €` |
| **Wi-Fi Professionnel** | 1 | Borne `HPE Aruba AP-505 (2)` | `1 017 €` | `1 017 €` |
| **Énergie / Onduleurs** | 1 | Onduleur `APC Smart-UPS X 3000VA (1)` | `2 000 €` | `2 000 €` |
| **Énergie / Onduleurs** | 1 | Onduleur `APC Smart-UPS X 3000VA (2)` | `2 000 €` | `2 000 €` |
| **Sécurité Périmétrique** | 1 | Pare-feu `Fortinet FortiGate 60F (1)` | `755 €` | `755 €` |
| **Sécurité Périmétrique** | 1 | Pare-feu `Fortinet FortiGate 60F (2)` | `755 €` | `755 €` |
| **Commutation Réseau** | 1 | Switch `Netgear GS752TXv3 (1)` (48 ports, liaisons 10G) | `600 €` | `600 €` |
| **Commutation Réseau** | 1 | Switch `Netgear GS752TXv3 (2)` (48 ports, liaisons 10G) | `600 €` | `600 €` |
| **Salle Serveur** | 1 | Baie `42U` verrouillable, PDU, kits rack `1U` et accessoires | `3 300 €` | `3 300 €` |
| **Licences Serveur** | 4 | `Windows Server Standard 2025` | `800 €` | `3 200 €` |
| **Licences Accès** | 25 | `Windows Server CAL` Utilisateurs | `44 €` | `1 100 €` |
| **TOTAL CAPEX** | — | — | — | **`40 844 € HT`** |

---

## Détail des dépenses de fonctionnement (OPEX)

| Catégorie | Contrat / Service récurrent | Prix / an HT |
| :--- | :--- | :---: |
| **Télécoms** | Fibre dédiée **FTTO** (`1 Gbit/s` symétrique, GTR 4h) | `6 500 €` |
| **Télécoms** | Fibre de secours **FTTH** (`1 Gbit/s`) | `750 €` |
| **Cloud & Stockage** | `AWS S3` (Object Lock 30j) + `AWS Glacier` | `1 200 €` |
| **Sauvegarde** | `Veeam Data Platform Premium` (x10 licences) | `2 400 €` |
| **Sauvegarde** | `Veeam Cloud Connect` (M365 + sauvegarde NAS) | `1 400 €` |
| **Support matériel** | `Dell ProSupport` (4 serveurs PowerEdge R350) | `1 500 €` |
| **Support matériel** | `HPE Aruba Foundation Care` (2 bornes AP-505) | `60 €` |
| **Support matériel** | `Netgear Overdrive Support` (2 switchs GS752TXv3) | `250 €` |
| **Support matériel / Sécurité** | Support & licences `FortiGate 60F` | `250 €` |
| **Support matériel** | Support `Smart-UPS X 3000VA` | `80 €` |
| **Support matériel** | Support `Synology RackStation RS2423+` | `250 €` |
| **Hébergement Web** | `OVHcloud Pro` (nom de domaine + hébergement Web) | `38 €` |
| **Bureautique & SaaS** | `Microsoft 365 Business Premium` (21 comptes / an : SharePoint, Exchange Online, Defender for Business, MFA) | `5 040 €` |
| **TOTAL OPEX** | — | **`19 718 € HT / an`** |

---

## Analyse budgétaire

- **Les deux premiers postes OPEX** représentent près de **58,5 %** des coûts récurrents annuels :
  1. **La fibre dédiée FTTO (`6 500 € / an`)** : indispensable pour garantir la continuité de la transmission en temps réel des sondes maritimes grâce à une GTR de 4 heures et une redondance avec la ligne FTTH.
  2. **Microsoft 365 Business Premium (`5 040 € / an`)** : mutualise à coût maîtrisé le remplacement du serveur de fichiers (`SharePoint` / `OneDrive`), la messagerie sécurisée (`Exchange Online`), l'EDR (`Defender for Business`) et la `MFA`, tout en supprimant le CAPEX et la maintenance de serveurs sur site supplémentaires.
- Les tarifs publics professionnels retenus constituent une enveloppe prudente pouvant être optimisée lors des négociations fournisseurs.
