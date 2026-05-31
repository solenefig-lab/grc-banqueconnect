# Timeline de la gestion de crise cyber — BanqueConnect

---

## Contexte

### BanqueConnect — Néobanque européenne inspirée d'acteurs comme Revolut, N26, Chime.

- 50 employés · 200 000 clients
- Application mobile B2C · Paiements instantanés · Virements SEPA · API Open Banking · WebApp de gestion de compte
Environnement technique
- Hébergement : AWS (France) · PRA/PCA OVHcloud
- Fournisseurs critiques : KYCPro (KYC offshore) · Stripe (paiements) · SendGrid (emails transactionnels)
- Outils internes : Slack · Jira · Zendesk

### Données sensibles

- Clients : IBAN · Emails · Téléphones · Historiques de transaction · SSN (clients US)
- Internes : Contrats · RH · Données stratégiques · Informations financières

### Référentiels applicables

- Prioritaires : DORA · RGPD · DSP2
- Complémentaires : ISO 27001:2022 · ISO 27005 · EBIOS RM · NIS2

### Parties prenantes

| Rôle | Responsable | Actions pendant la crise | Conflits d’intérêts |
|----------|-----------------|-------------------------------|----------------------|
| **COMEX** | CEO | Prise de décision | **Business** : Maintenir les services vs **Sécurité** : Limiter les risques. |
| **SOC** | Responsable SOC (N3)| Investigation technique (logs, forensic) | **Urgence** : Agir vite vs **Précision** : Attendre les preuves. |
| **RSSI** | RSSI | Coordination de la cellule de crise · Arbitrages techniques · Recommandations au COMEX | **Sécurité** : Coupure API vs **Business** : Continuité de service |
| **Juridique** | DPO | Notification ACPR, gestion des litiges | **Conformité** : Respecter les délais vs **Risque juridique** : limiter les sanctions |
| **Communication** | Directeur Com | Communiqué de presse, FAQ clients |  **Transparence** : Informer les clients vs **Protection** : Éviter la panique. |
| **Juridique** | Directeur Juridique | Gestion des aspects légaux et contractuels | **Contractuel**: résilier immédiatement KYCPro  vs. **Opérationnel** : négocier une transition (risque de récidive) ? |
| **Compliance** | Responsable Compliance | Vérification des obligations DORA, notification ACPR | **Réglementaire** : Respecter DORA vs **Opérationnel** : Ne pas bloquer les processus. |
| **DPO** | Data Protection Officer | Notification CNIL, notification aux personnes concernées, tenue du registre des traitements, conseil sur la licéité des traitements | Protection des droits des personnes vs calendrier de communication maîtrisé par le Juridique.|


---

## 1 - Phase de détection (J-3 à J-1)

| Jour/Heure | Événement | Actions | Propriétaire | Statut | Détails |
|---|---|---|---|---|---|
| J-3 14:00 | Premières anomalies : latence accrue sur l'API Open Banking (+200ms). | Vérification des métriques · Hypothèse initiale : problème réseau ou charge anormale. | SOC (Analyste L1) | ⚠️ Fausse alerte — classé incident mineur. | Outils : Prometheus, Grafana. Logs : `api-openbanking-latency.log` |
| J-3 16:00 | Alertes SOC : connexions suspectes depuis une IP inconnue (185.220.101.x) sur `/api/kyc/verify`. | Analyse des logs : 5 req/min depuis 185.220.101.x (vs 0 habituellement) · Vérification IP : géolocalisation Russie (VPN) · Classement : incident de sécurité niveau moyen. | SOC (Analyste L2) | ⚠️ En investigation. | Outils : Splunk, VirusTotal. Action : blocage temporaire de l'IP. |
| J-2 09:00 | Escalade SOC : 185.220.101.x tente d'exfiltrer des données via `/api/kyc/export`. | Forensic : analyse des payloads (requêtes POST suspectes) · Hypothèse : exploitation d'une faille KYCPro · Alerte envoyée au RSSI et à KYCPro. | SOC (Analyste L3) + RSSI | ⚠️ Investigation approfondie. | Outils : Wireshark, Burp Suite. Preuve : `api-kyc-export.log` |
| J-2 14:00 | Réponse KYCPro : "Aucune anomalie détectée de notre côté." | Demande de logs KYCPro pour vérification · KYCPro promet de fournir les logs sous 24h. | SOC + KYCPro | ❌ Retard dans l'investigation. | Fournisseur tiers minimise l'incident. |
| J-2 18:00 | Nouvelle alerte : augmentation des erreurs 500 sur `/api/payments` (lié à KYCPro). | Corrélation : erreurs 500 coïncident avec connexions 185.220.101.x · Hypothèse : injection SQL ou abus d'API. | SOC + DevOps | ⚠️ Incident critique suspecté. | Outils : SIEM (corrélation des logs). Action : désactivation temporaire de `/api/kyc/export`. |

---

## 2 - Compromission confirmée (J-1)

| Jour/Heure | Événement | Actions | Propriétaire | Statut | Détails |
|---|---|---|---|---|---|
| J-1 10:00 | KYCPro admet une intrusion : "Une faille dans notre API (CVE-2026-4242) a été exploitée." | Demande d'un rapport technique à KYCPro · Préparation d'un accord de confidentialité pour l'échange d'informations. | SOC + KYCPro + Juridique | ⚠️ Confirmation partielle. | CVE fictive : CVE-2026-4242 (faille d'authentification KYCPro). |
| J-1 12:00 | Preuves d'exfiltration : requêtes POST anormales vers un serveur externe (IP : 45.67.89.12). | Forensic : analyse des payloads → données exfiltrées : IBAN, emails, SSN (5 000 clients). | SOC + RSSI | 🔴 Incident critique suspecté. | Outils : Zeek, Volatility. Incident critique suspecté mais qualification formelle non actée : périmètre forensic incomplet, instance dirigeante non encore réunie. Délai de qualification assumé jusqu'à obtention d'éléments suffisants. Documenté en cellule de crise (voir CR cellule de crise J-1 — présupposé, non livré dans ce projet). |
| J-1 14:00 | Réunion SOC + RSSI + Juridique : fuite de données confirmée via KYCPro. | Décision : isoler KYCPro de l'API Open Banking · Alerte interne au COMEX et à la Communication · Cellule de crise activée. | SOC + RSSI + Juridique | ✅ Isolation KYCPro. | Action concrète : désactivation du token API KYCPro. |
| J-1 16:00 | Revendication "Not42" sur Telegram : "BanqueConnect, vos données KYC sont entre nos mains." + échantillon (10 IBAN). | Vérification SOC : les IBAN correspondent à des clients BanqueConnect · Classification : incident majeur + menace de fuite publique. | SOC + Juridique | 🔴 Crise médiatique imminente. | Preuve : capture d'écran Telegram. Action : ne pas répondre à Not42 (recommandation ANSSI). |

> À J-1 10:00, KYCPro confirme la compromission officiellement. À J-1 12:00, le SOC et le RSSI confirment que les données de BanqueConnect sont impactées — l'incident critique est suspecté mais non encore qualifié formellement. À J-1 14:00, la cellule de crise décide de la première action de mitigation.

---

## 3 - Phase de gestion de crise (J-0)

| Jour/Heure | Événement | Actions | Propriétaire | Statut | Détails |
|---|---|---|---|---|---|
| J-0 08:00 | Article dans Les Échos : "BanqueConnect : suspicion de fuite de données clients." | Préparation d'un communiqué de crise · Vérification des obligations DORA (délai 4h). | Communication + Juridique | 🔴 Crise médiatique. | Source : journaliste a contacté le service presse. Action : ne pas confirmer avant validation interne. |
| J-0 09:00 | Réunion COMEX d'urgence. | Options discutées : 1. Couper l'API Open Banking (−2M€/jour) · 2. Maintenir l'API avec surveillance renforcée · 3. Notifier l'ACPR sous 4h. Décision : maintenir l'API + surveillance renforcée + notification ACPR sous 4h. | COMEX (CEO + RSSI + DPO + Directeur Com) | ✅ Décision prise. | Matrice d'arbitrage et compte-rendu COMEX ci-dessous. |
| J-0 10:00 | Notification interne : email aux employés "Incident de sécurité en cours. Suivez les consignes du SOC." | Cible : tous les employés · Message : transparent mais rassurant. | Communication interne | ✅ Envoyé. | Template communication interne. |
| **J-0 11:00** | **Début officiel de la gestion de crise (aligné DORA/RGPD). Confirmation exfiltration. Classement incident majeur.** | Activation cellule de crise (SOC + Juridique + Communication + COMEX) · Procédure DORA : délai 4h pour notifier l'ACPR · Procédure RGPD : préparation notification clients (72h). | RSSI (Coordinateur) | ✅ Lancé. | Qualification formelle actée après COMEX J-0 09:00. Délai J-1 12:00 → J-0 11:00 justifié : forensic incomplet, périmètre provisoire, instance compétente non réunie avant J-0. Point susceptible d'être questionné par l'ACPR — justification documentée. |
| J-0 12:00 | Notification à l'ACPR (DORA Art. 19). | Contenu : description de l'incident · données concernées (IBAN, SSN, emails) · mesures prises (isolation KYCPro, audit en cours) · délai : 1h après qualification formelle. | RSSI | ✅ Envoyé. | Respect du délai 4h DORA. |
| J-0 14:00 | Communiqué de presse : "BanqueConnect a détecté une anomalie de sécurité. Investigation en cours." | Ton : transparent, sans détails techniques · Cible : clients, médias, partenaires. | Communication externe | ✅ Publié. | - |
| J-0 16:00 | Premières réactions clients : afflux de demandes sur Zendesk (+300%). | FAQ urgente publiée · Message : "Vos fonds sont sécurisés. Changez votre mot de passe par précaution." | Support Client | ✅ FAQ en ligne. | Mise à jour FAQs. |

---

### Compte-rendu COMEX avec matrice d’arbitrage (business vs sécurité).

# Compte-rendu Réunion COMEX — J-0 (29/05/2026, 09:00)
**Participants** : CEO, RSSI, Compliance , Directeur Communication, Juridique, DPO.

**Contexte** :
- **Incident** : Compromission de KYCPro → Exfiltration de données clients (IBAN, SSN, emails).
- **Revendication** : Groupe hacktiviste **Not42** (Telegram).
- **Médiatisation** : Article dans *Les Échos* (08:00).

**Options discutées** :

| Option | Avantages | Risques | Impact Business | Délai | Décision |
| --- | --- | --- | --- | --- | --- |
| Couper l’API Open Banking | Sécurité maximale | Perte de revenus (-2M€/jour) | 🔴 Élevé | Immédiat | ❌ **Non** (mais **le RSSI a émis une réserve formelle** : "Risque inacceptable de nouvelle fuite. Je recommande la coupure.") |
| Maintenir l’API avec surveillance renforcée | Équilibre sécurité/business | Risque de nouvelle fuite | ⚠️ Moyen | Immédiat | ✅ Oui |
| Notifier l’ACPR sous 4h (DORA Art. 19) | Conformité DORA | Risque de sanctions si retard | 🟢 Faible | 4h | ✅ Oui , anticipation de la qualification probable; principe de précaution, en assumant que la qualification formelle interviendra dans l'heure.|
| Notifier la CNIL sous 72h (RGPD Art. 33) | Conformité RGPD | Risque de sanctions (4% CA) | 🟢 Faible | 72h | ✅ Oui |
|   Notifier les clients sous 72h (RGPD Art. 34) | Transparence, confiance | Risque de panique | ⚠️ Moyen | Sous **72h (meilleur délai possible)** | ✅ Oui **Justification** : Risque élevé (IBAN + SSN exposés → risque de fraude/usurpation). |
| Communiquer immédiatement (communiqué de presse) | Transparence | Risque de panique clients | ⚠️ Moyen | J-0 14:00 | ✅ Oui; Communication : "Il faut communiquer vite pour rassurer." Juridique : "Attendons d’avoir plus d’infos pour éviter des erreurs." → Arbitrage CEO : "On publie un communiqué générique à 14h, sans détails techniques." |


**Décisions finales** :
1. **Maintenir l’API Open Banking** avec :
   - **Surveillance renforcée** (logs en temps réel).
   - **Rate-limiting** sur les endpoints sensibles.
2. **Notifications réglementaires** :
   - **ACPR** : 
        - Notification sous **4h** (DORA Art. 19) → **J-0 12:00**.
        - Rapport intermédiaire **sous 72h** (DORA Art. 19.3) → **J+2 12:00**.
        - Rapport final sous 1 mois (DORA Art. 19.3) → J+30 (approx.).
   - **CNIL** : Notification sous **72h** (RGPD Art. 33) → **J+1 10:00**.
   - **Clients** : Notification sous **72h** (RGPD Art. 34) → **J+1 14:00**.
3. **Communication externe** :
   - **Communiqué de presse** (ton rassurant, sans détails techniques) → **J-0 14:00**.
   - **FAQ clients** → **J-0 16:00**.

**Prochaines étapes** :
- **J-0 11:00** : Début officiel de la gestion de crise (cellule de crise activée). - RSSI
- **J-0 12:00** : Envoi de la notification à l’ACPR. - Compliance + RSSI
- **J-0 14:00** : Publication du communiqué de presse. - Communication
- **J-0 16:00** : Mise à jour FAQs clients (et procédure support client) - Communication + Juridique
- **J+1 10:00** : Envoi de la notification à la CNIL. - DPO
- **J+1 14:00** : Envoi de la notification aux personnes concernées. - Communication + DPO

> Note: voir présentation COMEX du RSSI [Présentation COMEX](./Presentation-COMEX.md)

---

## 4 - Phase Post-Crise (J+1 à J+7)

| Jour/Heure | Événement | Actions | Propriétaire | Statut | Détails |
|---|---|---|---|---|---|
| J+1 09:00 | Audit KYCPro : rapport préliminaire → faille corrigée (CVE-2026-4242). | Patch déployé sur les serveurs KYCPro · Révocation des tokens compromis · Première estimation : 5 000 clients impactés (IBAN, emails, SSN). | SOC + KYCPro | ✅ Faille corrigée. | Preuve : rapport audit KYCPro J+1. Note : KYCPro minimise toujours l'ampleur de l'incident. |
| J+1 10:00 | Envoi notification CNIL (RGPD Art. 33). | Contenu : faille KYCPro (CVE-2026-4242), 5 000 clients (IBAN, emails, SSN), mesures prises (isolation, audit en cours) · Délai : 72h après confirmation (J-0 11:00 → J+1 10:00). | DPO | ✅ Envoyé. | Template défini dans procédure gestion de crise et notification. |
| J+1 14:00 | Notification clients — 1ère vague (RGPD Art. 34). | Cible : 5 000 clients · Contenu : données exposées (IBAN, emails), mesures prises, recommandation changement de mot de passe · Canal : email. | Communication + DPO | ✅ Envoyé. | Template défini dans procédure gestion de crise et notification. |
| J+2 09:00 | Débriefing à chaud (chronologie, premières observations). | Points abordés : retard KYCPro · détection tardive (46h) · faille CVE-2026-4242 · Actions immédiates : maintien surveillance APIs · évaluation conformité DORA des contrats fournisseurs · REX complet prévu à J+30. | COMEX + SOC + Juridique | ✅ Débriefing documenté. | - |
| J+2 10:00 | Nouvelle découverte forensic : 7 000 clients supplémentaires impactés. | Cause : accès non détectés entre J-5 et J-3 · Nouveau périmètre : 12 000 clients (vs 5 000) · Données supplémentaires : historiques de transaction (2 000 clients). | SOC + Forensic (cabinet externe) | 🔴 Incident élargi. | Preuve : rapport forensic J+2. Note : KYCPro conteste ces résultats. |
| J+2 12:00 | Rapport intermédiaire ACPR (DORA Art. 19.2). | Contenu : chronologie mise à jour (12 000 clients) · causes racines (faille KYCPro + délai de détection) · mesures correctives (patch, audit, monitoring) · Délai respecté : 72h après qualification (J-0 11:00 → J+2 12:00). | Compliance | ✅ Envoyé. | - |
| J+2 14:00 | Réunion COMEX : gestion du périmètre élargi. | Décisions : notification 7 000 nouveaux clients sous 72h (RGPD) · 2ème communiqué de presse · évaluation remplacement KYCPro (Onfido/Sumsub) · Tensions : Juridique : "On doit notifier sous 72h." Communication : "Un 2ème communiqué va relancer la panique." COMEX : "On notifie, mais on maîtrise le message." | COMEX + Juridique + Communication | ✅ Décision prise. | Compte-rendu COMEX J+2. |
| J+3 09:00 | 2ème vague de notifications clients (RGPD Art. 34). | Cible : 7 000 nouveaux clients · Contenu : données exposées (IBAN, emails, historiques de transaction), mesures prises, renforcement des contrôles · Canal : email. | Communication + DPO | ✅ Envoyé. | Note : 30% des clients contactent le support. |
| J+3 10:00 | 2ème communiqué de presse (transparence sur l'élargissement). | Contenu : mise à jour périmètre (12 000 clients vs 5 000) · mesures de sécurité renforcées · Ton : transparent mais rassurant. | Communication | ✅ Publié. | - |
| J+3 11:00 | Mise à jour notification CNIL (RGPD Art. 33). | Contenu : extension périmètre à 12 000 clients · ajout historiques de transaction (2 000 clients) · Justification : nouvelle faille CVE-2026-4243 confirmée par Mandiant · Délai : sous 72h après découverte. | DPO | ✅ Envoyé. | Notification complémentaire conforme RGPD. |
| J+3 14:00 | KYCPro conteste le rapport forensic : "Les 7 000 clients supplémentaires ne sont pas liés à notre faille." | Réanalyse des logs avec Mandiant · Préparation recours contractuel contre KYCPro · Menace de résiliation si KYCPro ne coopère pas. | SOC + Juridique + COMEX | ⚠️ Conflit en cours. | Email KYCPro : "Nous contestons votre analyse et exigeons une contre-expertise." |
| J+4 09:00 | Contre-expertise forensic (cabinet Mandiant). | Résultats : confirmation 12 000 clients impactés (dont 7 000 via 2ème faille non corrigée) · Nouvelle faille : CVE-2026-4243 (non patchée par KYCPro). | SOC + Mandiant | 🔴 Nouvelle faille détectée. | Preuve : rapport contre-expertise Mandiant. |
| J+4 11:00 | Réunion COMEX : décision sur KYCPro. | Options : maintenir KYCPro avec pénalités vs remplacer par Onfido (50k€/an) · Décision : remplacement sous 30 jours · Tensions : Juridique : "Résilier pour manquement." RSSI : "Migration = 2 mois de travail." COMEX : "On résilie avec délai de transition négocié." | COMEX + Juridique + DevOps | ✅ Décision prise. | Compte-rendu COMEX J+4. |
| J+4 14:00 | Début de la migration vers Onfido. | Contrat signé avec Onfido (clauses DORA incluses) · Tests des APIs Onfido · Formation des équipes aux nouvelles procédures KYC. | DevOps + Onfido | 🔄 En cours. | Échéance : migration complète sous 30 jours. |
| J+5 10:00 | KYCPro envoie une mise en demeure : "Vos allégations sont diffamatoires. Nous engageons des poursuites." | Préparation de la défense (preuves forensiques) · Décision COMEX : ne pas commenter le litige dans les communications futures. | Juridique + COMEX | ⚠️ Litige en cours. | Preuve : email de mise en demeure KYCPro. |
| **J+6 09:00** | **Clôture provisoire de la cellule de crise.** | Bilan : 12 000 clients notifiés, aucune nouvelle exfiltration détectée · Désactivation cellule de crise (veille renforcée maintenue) · Plan de transition : migration Onfido + audit de tous les fournisseurs. | COMEX | ✅ Clôture actée. | - |
| J+7 14:00 | Réunion de clôture avec KYCPro (dernière tentative de médiation). | Preuves Mandiant présentées : 2 failles confirmées · KYCPro refuse de reconnaître CVE-2026-4243 · Décision : poursuites judiciaires engagées. | Juridique + COMEX + KYCPro | ⚠️ Litige persistant. | KYCPro quitte la réunion sans accord. |
| J+30 | Rapport final ACPR + REX complet. | Périmètre final consolidé · causes racines · plan correctif. | Compliance + RSSI + DPO | 🔄 Prévu. | Voir section 5 — REX J+30. |


---

## 5 - Retour d'expérience (REX - J + 30)


## 📌 Contexte du REX
- **Date** : J + 30
- **Participants** : COMEX + RSSI + Juridique + Communication + DPO + Compliance
- **Objectif** : Analyser les **causes racines**, les **décisions prises**, et les **améliorations à apporter**.

---

## 🔴 Problèmes Identifiés

| **Problème** | **Cause** | **Impact** | **Responsable** |
|--------------|-----------|------------|-----------------|
| **Retard de KYCPro** | Manque de transparence + délai de 24h pour fournir les logs. | Retard dans la détection et la notification ACPR. | KYCPro |
| **Contestation juridique de KYCPro** | KYCPro a refusé de reconnaître la 2ème faille (CVE-2026-4243) et menacé de poursuites pour diffamation. | Litige persistant (risque de procès long et coûteux) + retard dans la résolution. | KYCPro + Juridique |
| **Dépendance à un seul fournisseur KYC** | Pas de backup pour KYCPro. KYCPro a minimisé l’incident pendant 48h. | Risque de blocage total du processus KYC + risque de récidive + perte de confiance dans la chaîne d’approvisionnement. | COMEX |
| **Manque de monitoring temps réel** | Alertes SOC non priorisées pour les endpoints KYC. | Détection tardive de l’exfiltration. | SOC |
| **Absence de clause contractuelle stricte** | Contrat KYCPro sans pénalité pour retard de notification. | Impossible de sanctionner KYCPro. | Juridique |
| **Communication interne désorganisée** | Pas de canal dédié pour les alertes crise. | Retards dans la prise de décision. | Communication |
 

---
## 🟢 Actions Correctives

| **Action** | **Description** | **Responsable** | **Échéance** | **Statut** | **Coût** |
|------------|-----------------|-----------------|--------------|------------|----------|
| **Remplacer fournisseur KYC KYCPro par Onfido** | Migration complète vers Onfido (contrat signé avec clauses DORA strictes : délai de réponse <4h, pénalités de 10k€/jour). KYCPro résilié pour manquement contractuel. |  COMEX + DevOps | J+30 | ✅ Terminé | 50k€/an |
| **Poursuites judiciaires contre KYCPro** | Engager un recours pour manquement aux obligations de sécurité (CVE-2026-4242 et CVE-2026-4243) et retard de notification. | Juridique | J+45 | 🔄 En cours | 30k€ (frais d’avocat) |
| **Déployer un SIEM dédié aux APIs** | **Splunk** ou **Elastic SIEM** pour détecter les exfiltrations en temps réel. | SOC | J+7 | ⚠️ En cours | 150k€/an |
| **Former les équipes à la gestion de crise** | Atelier **"Gestion des incidents avec fournisseurs récalcitrants"** (2h). | RH | J+14 | 🔄 Prévu | 10k€ |
| **Créer un canal Slack dédié aux alertes crise** | `#incident-cyber` avec **@mentions automatiques** pour le COMEX. | Communication | J+3 | ✅ Fait | - |
| **Audit de tous les fournisseurs critiques** | Vérifier les **clauses DORA** et les **plans de continuité**. | Compliance + Juridique | J+14 | 🔄 Prévu | 20k€ |

---
## 💡 Leçons Apprises

### 🔹 Pour la Direction (COMEX)
- **"Un incident cyber n’est pas qu’un problème technique"** :
  - **Impact business** : La décision de maintenir l’API a évité une perte de **2M€/jour**, mais a **augmenté le risque sécurité**.
  - **Leçon** : **Équilibrer sécurité et business** est un **arbitrage permanent** en crise.

- **"La transparence paie"** :
  - Le **communiqué de presse rapide** (J-0 14:00) a **limité la panique** malgré des informations incomplètes.
  - **Leçon** : En crise, **mieux vaut communiquer tôt avec peu d’infos que tard avec trop de détails**.

### 🔹 Pour le SOC
- **"Les fournisseurs tiers sont le maillon faible"** :
  - **KYCPro a minimisé l’incident** pendant 24h → **Retard critique**.
  - **Leçon** : **Ne jamais faire confiance aveuglément** à un fournisseur. **Vérifier par soi-même**.

- **"Le monitoring doit être proactif"** :
  - Les **alertes SOC** sur `/api/kyc/export` n’étaient pas **priorisées**.
  - **Leçon** : **Classer les endpoints critiques** et **ajuster les seuils d’alerte**.

### 🔹 Pour le Juridique
- **"DORA et RGPD sont complémentaires"** :
  - **DORA** (ACPR) → **4h/72h/1 mois**.
  - **RGPD** (CNIL) → **72h**.
  - **Leçon** : **Les deux doivent être respectés**, mais avec des **délais différents**.

- **"Les contrats doivent inclure des clauses DORA"** :
  - **KYCPro n’avait pas d’obligation de notification rapide**.
  - **Leçon** : **Exiger des clauses strictes** (délais, pénalités, audits).

### 🔹 Pour la Communication
- **"Un communiqué trop technique = panique"** :
  - Le **premier communiqué** (J-0 14:00) était **volontairement vague** → **moins de panique**.
  - **Leçon** : **Adapter le langage** au public (clients vs régulateurs).

- **"Préparer les FAQ à l’avance"** :
  - La **FAQ clients** a **réduit les appels au support** de 40%.
  - **Leçon** : **Anticiper les questions** pour désamorcer la crise.

---
## 📌 Recommandations pour l’Avenir
1. **Créer un "Playbook de Crise Cyber"** :
   - **Checklist** des actions à mener (qui fait quoi, quand, comment).
   - **Templates** de notifications (ACPR, CNIL, clients).
   - **Arbre décisionnel** (ex. : "Si nouvelle fuite → couper l’API").

2. **Simuler des exercices de crise** :
   - **Scénario** : "Compromission d’un fournisseur KYC".
   - **Objectif** : Tester la **réactivité** et la **coordination** entre équipes.

3. **Exiger des audits tiers indépendants** :
    - Pour tous les fournisseurs critiques : Audit annuel par un cabinet externe (ex. : Mandiant, Wavestone).
    - Coût : 50k€/an/fournisseur.

4. **Renforcer la collaboration avec les régulateurs** :
   - **ACPR** : Point de contact dédié.
   - **CNIL** : Procédure de notification pré-approuvée.


---

## Principaux livrables associés

| Phase | Livrable | Contenu  | Exemple |
| --- | --- | ----------- | --- | 
| Détection | Logs API Open Banking | Extrait des logs montrant la latence et les connexions suspectes. | - |
| Détection | Rapport SOC – J-2 | Synthèse des alertes et hypothèses. | - |
| Détection | Email d’escalade au RSSI | Alerte envoyée par le SOC au RSSI. | - |
| Compromission confirmée | Email de KYCPro | Confirmation de l’intrusion. | - |
| Compromission confirmée | Preuves d’exfiltration | Échantillon de données fuitées. | - |
| Compromission confirmée | Message Telegram de Not42 | Capture d’écran de la revendication. | - |
| Compromission confirmée | Compte-rendu réunion SOC | Décision d’isoler KYCPro. | - |
| Gestion de crise | Comptes-rendus COMEX | Décisions clés (maintenir l’API, notifier ACPR). | voir ci-dessus (3 - Gestion de crise / Compte-rendu COMEX) et [Présentation COMEX](./Presentation-COMEX.md)|
| Gestion de crise | Notification initiale ACPR | Template conforme à DORA Art. 19. | Template défini dans procédures gestion de crise BanqueConnect |
| Gestion de crise | Communiqués de presse | Message public transparent. | - |
| Gestion de crise | FAQ clients | Réponses aux questions fréquentes. | - |
| Post-Crise | Audit KYCPro J+1 | Rapport préliminaire → Faille corrigée (CVE-2026-4242). | - |
| Post-Crise | Rapport forensic complet – J+2 | Découverte de la 2ème faille (CVE-2026-4243) → 12 000 clients impactés. | - |
| Post-Crise | Notification CNIL (1ere et mise à jour)| Notification conforme à CNIL/RGPD | Exemple SantéConnect [Procédure Incident Notification RGPD] (https://github.com/solenefig-lab/grc-pme-fictive/blob/main/semaine-2-rgpd-hds/procedure-incident-notification.md) |
| Post-Crise | Notification aux clients (1ere et 2eme vague, RGPD) | Notification conforme à CNIL/RGPD | Exemple SantéConnect [Procédure Incident Notification RGPD (https://github.com/solenefig-lab/grc-pme-fictive/blob/main/semaine-2-rgpd-hds/procedure-incident-notification.md) |
| Post-Crise | Rapport intermédiaire ACPR (DORA Art. 19). | Chronologie + causes + mesures + plan action amélioration contiue | Template défini dans procédures gestion de crise BanqueConnect |
| Post-Crise | Contre-expertise Mandiant – J+4 | Confirmation des 12 000 clients impactés + 2ème faille (CVE-2026-4243). | - |
| Post-Crise | Débriefing à chaud | 12 000 clients notifiés, 0 nouvelle fuite, migration vers Onfido démarrée. | - |
| J + 30 | Rapport final ACPR  | Périmètre final (12 000 clients), causes racines, mesures correctives. |- |
| J + 30 | Rapport Retour d’Expérience – J+30 | Causes racines, actions correctives, leçons apprises. | voir ci-dessus (5 - Retour d'Expérience) |

> Note: dans la colonne Exemple, - indique que le livrable est présupposé dans le cadre de la gestion de crise mais n'est pas produit dans ce projet. Certains livrables renvoient vers des exemples issus d'un projet distinct.