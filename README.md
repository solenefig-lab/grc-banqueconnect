# Gestion de crise cyber — BanqueConnect

![Domaine](https://img.shields.io/badge/Domaine-GRC%20%7C%20Cybersécurité-purple)
![Secteur](https://img.shields.io/badge/Secteur-bancaire-blue)
![Référentiels](https://img.shields.io/badge/Référentiels-RGPD%20%7C%20DORA%20%7C%20NIS2%20%7C%20ISO%2027001-red)
![Status](https://img.shields.io/badge/Status-En%20cours-orange)
![Niveau](https://img.shields.io/badge/Niveau-Midlevel%20GRC-yellow)

> ⚠️ **Projet de démonstration pédagogique** — BanqueConnect est une néobanque européenne fictive confrontée à une cybercrise majeure. Tous les noms, données et événements sont fictifs.
> Les livrables reflètent une démarche de gestion de crise réelle adaptée à un contexte simulé, inspiré d'actualités récentes (SolarWinds, Okta, 3CX) où des fournisseurs tiers ont **minimisé ou retardé** la communication d'incidents.

---

## Contexte

**BanqueConnect** est une néobanque européenne (inspirée de Revolut, N26, Chime) confrontée à une exfiltration de données clients via un fournisseur tiers (KYCPro).

- 50 employés · 200 000 clients
- Services : application mobile B2C · paiements instantanés · virements SEPA · API Open Banking
- Hébergement : AWS (France) · PRA/PCA OVHcloud
- Fournisseurs critiques : KYCPro (KYC offshore) · Stripe (paiements) · SendGrid (emails)
- Outils internes : Slack · Jira · Zendesk

### Données sensibles

| Type | Exemples |
|---|---|
| Clients | IBAN · emails · téléphones · historiques de transaction · SSN (clients US) |
| Internes | Contrats · RH · données stratégiques · informations financières |

### Référentiels applicables

- Prioritaires : DORA · RGPD · DSP2
- Complémentaires : ISO 27001:2022 · ISO 27005 · NIS2

### Parties prenantes

| Rôle | Responsable | Actions pendant la crise | Conflits d'intérêts |
|---|---|---|---|
| **COMEX** | CEO | Prise de décision finale | Business : maintenir les services vs Sécurité : limiter les risques |
| **SOC** | Responsable SOC (N3) | Investigation technique : logs, forensic, analyse des IOC | Urgence : agir vite vs Précision : attendre les preuves |
| **RSSI** | RSSI | Coordination cellule de crise · arbitrages techniques · recommandations COMEX · réserve formelle documentée | Sécurité : coupure API vs Business : continuité de service |
| **DPO** | Data Protection Officer | Notification CNIL · notification clients · registre des traitements | Protection des droits des personnes vs calendrier de communication maîtrisé par le Juridique |
| **Compliance** | Responsable Compliance | Vérification obligations DORA · notification ACPR | Réglementaire : respecter DORA vs Opérationnel : ne pas bloquer les processus |
| **Juridique** | Directeur Juridique | Gestion aspects légaux et contractuels · litiges KYCPro | Contractuel : résilier KYCPro vs Opérationnel : négocier une transition |
| **Communication** | Directeur Com | Communiqués de presse · FAQ clients | Transparence : informer vs Protection : éviter la panique |
| **Support Client** | Équipe Support | Gestion demandes clients (Zendesk) | Réactivité : répondre vite vs Précision : éviter les erreurs |

---

## Livrables

| Livrable | Description |
|---|---|
| [Timeline gestion de cybercrise](./timeline-crise.md) | J-3 à J+30 · cellule de crise · CR COMEX · matrice d'arbitrage · REX |
| [Présentation COMEX — RSSI](./Presentation-COMEX.md) | Simulation présentation en séance J+0 · slides épurées + notes personnelles du présentateur |

> **Note :** Dans la colonne Exemple de la timeline, `-` indique que le livrable est présupposé dans le cadre de la gestion de crise mais n'est pas produit dans ce projet. Certains livrables renvoient vers des exemples issus d'un projet distinct.

---

## Objectifs pédagogiques

- Simuler une gestion de crise cyber réaliste pour une néobanque (obstacles, arbitrages, tensions entre équipes)
- Appliquer les référentiels bancaires (DORA, RGPD, DSP2, ISO 27001, NIS2) dans un contexte opérationnel
- Traduire les exigences réglementaires en actions concrètes (matrices d'arbitrage, templates de notifications)
- Coordonner une cellule de crise multi-équipes avec rôles clairs et tensions documentées

---

## À propos

**Solène Figueiredo** — Consultante GRC | Conformité RGPD · Gestion des risques · Interface IT-Métier

Positionnement : interface entre technique, risques et décision métier, au service de projets cybersécurité ancrés dans la réalité opérationnelle.

**Parcours :** 10 ans de pilotage de projets digitaux internationaux chez Renault — campagnes simultanées dans plus de 100 pays (Twizy), programmes CRM multi-marchés (ZOE), gestion de plateformes digitales sur 40 pays.

**Aujourd'hui :** spécialisation en cybersécurité avec un positionnement opérationnel en GRC, gestion des risques et coordination de projets sécurité.

[![LinkedIn](https://img.shields.io/badge/LinkedIn-solenefigueiredo-blue)](https://www.linkedin.com/in/solene-figueiredo/)