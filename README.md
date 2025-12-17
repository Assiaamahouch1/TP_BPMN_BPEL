# TP1 - BPM & BPEL
## Process Mining - ENSA Khouribga 2025-2026

---

**Réalisé par :**  
- AMAHOUCH Assia  
- AATIQ Sawssan

**Filière :** IID 3ème année  
**Date :** Décembre 2025  
**Module :** Process Mining (PM)

---

## 📋 Table des matières

1. [Introduction](#-introduction)
2. [Environnement de travail](#-environnement-de-travail)
3. [Partie 1 : Modélisation BPM avec Bonita](#-partie-1--modélisation-bpm-avec-bonita)
4. [Partie 2 : Analyse BPEL/WSDL](#-partie-2--analyse-bpelwsdl)
5. [Résultats et tests](#-résultats-et-tests)
6. [Conclusion](#-conclusion)

---

## 🎯 Introduction

### Objectif du TP
Ce travail pratique vise à maîtriser les outils de modélisation et d'orchestration de processus métiers. Il comprend deux parties indépendantes :

1. **Modélisation BPM** : Création d'un processus d'achat d'article avec Bonita Studio
2. **Analyse BPEL/WSDL** : Extraction des composants d'un service web

### Cas d'étude
Nous avons modélisé un processus d'achat d'article avec trois modes de paiement :
- **Carte bancaire (CB)** avec sous-processus
- **Chèque** avec vérification CIN
- **Espèce** avec rendu de monnaie

---

## 💻 Environnement de travail

### Outils installés

| Outil | Version | Usage |
|-------|---------|-------|
| Bonita Community | 2024.1 | Modélisation et exécution des processus |
| JDK | OpenJDK 17 | Environnement Java requis |
| Navigateur | Chrome/Firefox | Accès au Bonita Portal |

### Installation de Bonita

```bash
# Vérification du JDK
java -version

# Téléchargement depuis
https://fr.bonitasoft.com/telechargez

# Extraction et lancement
./BonitaStudioCommunity
```

**Interface de démarrage :**
```
┌─────────────────────────────────────────┐
│     Bonita Studio Community 2024.1      │
│                                         │
│  [Nouveau]  [Ouvrir]  [Importer]       │
│                                         │
│  Diagrammes récents :                   │
│  - Achat_Article                        │
│  - Paiement_Par_Carte                   │
└─────────────────────────────────────────┘
```

---

## 🔄 Partie 1 : Modélisation BPM avec Bonita

### 1.1 Analyse du processus

**Flux principal du processus d'achat :**

```
Début
  ↓
Choisir mode de paiement (CB / Chèque / Espèce)
  ↓
┌─────────────┬─────────────┬─────────────┐
│   Si CB     │  Si Chèque  │  Si Espèce  │
│      ↓      │      ↓      │      ↓      │
│  Paiement   │  Vérifier   │  Rendre     │
│  par carte  │     CIN     │  monnaie    │
└─────────────┴─────────────┴─────────────┘
  ↓
Remettre l'article
  ↓
Fin
```

### 1.2 Diagramme BPMN créé

**Processus principal : Achat_Article**

```
    ┌─────┐
    │START│
    └──┬──┘
       │
    ┌──▼──────────────────────┐
    │  Choisir mode paiement  │  ← Human Task
    │  (CB/Chèque/Espèce)     │
    └──┬──────────────────────┘
       │
    ┌──▼──┐
    │ ◇XOR│  ← Gateway Exclusif
    └┬─┬─┬┘
     │ │ │
  ┌──▼ ▼ ▼──┐
  │CB │Chq│Esp│
  │   │   │   │
┌─▼─┐ │   │
│Call│ │   │  ← Call Activity
│Act.│ │   │     (Paiement_Par_Carte)
└─┬─┘ │   │
  │ ┌─▼───────────┐ ┌─▼──────────────┐
  │ │ Vérifier CIN│ │ Rendre monnaie │
  │ └─┬───────────┘ └─┬──────────────┘
  └───┴───────────────┴────┘
              │
      ┌───────▼────────────┐
      │ Remettre l'article │
      └───────┬────────────┘
              │
          ┌───▼───┐
          │  END  │
          └───────┘
```

### 1.3 Configuration des éléments BPMN

#### Gateway Exclusif (XOR)
```groovy
// Variable de processus
String modePaiement; // Valeurs possibles: "CB", "Cheque", "Espece"

// Conditions sur les transitions
Transition vers CB     : modePaiement == "CB"
Transition vers Chèque : modePaiement == "Cheque"
Transition vers Espèce : modePaiement == "Espece"
```

#### Annotations ajoutées
```
📝 "Sélection du mode de paiement selon le choix du client"
   → Sur le Gateway XOR

📝 "Vérification obligatoire pour les paiements par chèque"
   → Sur l'activité "Vérifier CIN"

📝 "Calcul automatique : montant reçu - prix article"
   → Sur l'activité "Rendre monnaie"
```

### 1.4 Sous-processus : Paiement par carte

**Diagramme : Paiement_Par_Carte**

```
┌─────┐     ┌───────────────┐     ┌─────────────┐     ┌────────────┐     ┌─────┐
│START│────→│ Insérer carte │────→│ Saisir code │────→│ Retirer CB │────→│ END │
└─────┘     └───────────────┘     └─────────────┘     └────────────┘     └─────┘
              Human Task            Human Task          Human Task
```

**Transformation en Call Activity :**
```
Dans le processus Achat_Article :
1. Sélectionner l'activité "Paiement CB"
2. Clic droit → Change type → Call Activity
3. Configuration :
   - Called process: Paiement_Par_Carte
   - Version: Latest
```

### 1.5 Configuration des formulaires

#### Formulaire "Choisir mode de paiement"
```
┌─────────────────────────────────────┐
│   Sélectionnez le mode de paiement  │
│                                     │
│   ⚪ Carte bancaire (CB)            │
│   ⚪ Chèque                          │
│   ⚪ Espèce                          │
│                                     │
│         [Valider]                   │
└─────────────────────────────────────┘
```

#### Formulaire "Vérifier CIN"
```
┌─────────────────────────────────────┐
│   Vérification du CIN               │
│                                     │
│   Numéro CIN: [____________]        │
│                                     │
│   Format: AB123456                  │
│                                     │
│         [Vérifier]                  │
└─────────────────────────────────────┘

Validation: Pattern ^[A-Z]{1,2}[0-9]{1,6}$
```

#### Formulaire "Rendre monnaie"
```groovy
// Script Groovy pour calcul automatique
def prixArticle = 150.00
def montantRecu = 200.00
def monnaieARendre = montantRecu - prixArticle

return monnaieARendre // Affiche: 50.00 DH
```

---

## 📡 Partie 2 : Analyse BPEL/WSDL

### 2.1 Fichier WSDL analysé

Le fichier WSDL fourni décrit un service web de réservation d'hôtel.

### 2.2 Extraction des composants

#### 1️⃣ Nom du service web
```xml
<service name="ReservationHotelService">
```
**Réponse :** `ReservationHotelService`

---

#### 2️⃣ Méthodes fournies
```xml
<portType name="ReservationHotel">
    <operation name="ChercherChambre">
    <operation name="ReserverChambre">
</portType>
```

**Réponse :**
| N° | Méthode | Description |
|----|---------|-------------|
| 1 | `ChercherChambre` | Recherche des chambres disponibles |
| 2 | `ReserverChambre` | Réservation d'une chambre |

---

#### 3️⃣ Messages d'entrées et de sorties

**ChercherChambre :**
```xml
<operation name="ChercherChambre">
    <input message="tns:ChercherChambre"/>
    <output message="tns:ChercherChambreResponse"/>
</operation>
```

**ReserverChambre :**
```xml
<operation name="ReserverChambre">
    <input message="tns:ReserverChambre"/>
    <output message="tns:ReserverChambreResponse"/>
</operation>
```

**Tableau récapitulatif :**

| Opération | Input | Output |
|-----------|-------|--------|
| ChercherChambre | `tns:ChercherChambre` | `tns:ChercherChambreResponse` |
| ReserverChambre | `tns:ReserverChambre` | `tns:ReserverChambreResponse` |

---

#### 4️⃣ Protocole de liaison
```xml
<binding name="ReservationHotelPortBinding" type="tns:ReservationHotel">
    <soap:binding transport="http://schemas.xmlsoap.org/soap/http" 
                  style="document"/>
```

**Réponse :** `SOAP over HTTP`

**Détails :**
- **Protocole de transport :** HTTP
- **Format de message :** SOAP (Simple Object Access Protocol)
- **Style :** Document/Literal

---

### 2.3 Synthèse de l'analyse

```
┌──────────────────────────────────────────────────────┐
│         Service Web: ReservationHotelService         │
├──────────────────────────────────────────────────────┤
│                                                      │
│  Opération 1: ChercherChambre                       │
│  ├─ Input:  tns:ChercherChambre                     │
│  └─ Output: tns:ChercherChambreResponse             │
│                                                      │
│  Opération 2: ReserverChambre                       │
│  ├─ Input:  tns:ReserverChambre                     │
│  └─ Output: tns:ReserverChambreResponse             │
│                                                      │
│  Protocole: SOAP over HTTP                          │
│  Style:     Document/Literal                        │
└──────────────────────────────────────────────────────┘
```

---

## ✅ Résultats et tests

### Tests effectués dans Bonita Portal

#### Test 1 : Paiement par Carte Bancaire
```
Scénario :
1. Démarrer le processus "Achat_Article"
2. Sélectionner "CB"
3. Le système appelle le sous-processus "Paiement_Par_Carte"
4. Compléter les 3 étapes :
   - ✓ Insérer carte
   - ✓ Saisir code (ex: 1234)
   - ✓ Retirer CB
5. Remettre l'article

Résultat : ✅ Processus terminé avec succès
```

#### Test 2 : Paiement par Chèque
```
Scénario :
1. Sélectionner "Chèque"
2. Saisir CIN : "AB123456"
3. Validation du format CIN
4. Remettre l'article

Résultat : ✅ Processus terminé avec succès
```

#### Test 3 : Paiement en Espèce
```
Scénario :
1. Sélectionner "Espèce"
2. Prix article : 150.00 DH
3. Montant reçu : 200.00 DH
4. Calcul automatique : 50.00 DH à rendre
5. Remettre l'article

Résultat : ✅ Processus terminé avec succès
Monnaie rendue : 50.00 DH
```

### Capture du Bonita Portal

```
┌──────────────────────────────────────────────────────────┐
│  Bonita Portal - Mes tâches disponibles                  │
├──────────────────────────────────────────────────────────┤
│                                                          │
│  ✓ Choisir mode de paiement      [Démarrer]            │
│  ✓ Vérifier CIN                  [Compléter]            │
│  ✓ Rendre monnaie                [Compléter]            │
│  ✓ Remettre l'article            [Compléter]            │
│                                                          │
│  Processus en cours : 3                                 │
│  Processus terminés : 12                                │
└──────────────────────────────────────────────────────────┘
```

---

## 🎓 Conclusion

### Compétences acquises

Au terme de ce TP, nous avons développé les compétences suivantes :

✅ **Modélisation de processus métiers**
- Maîtrise de la notation BPMN 2.0
- Utilisation des gateways, activités et événements
- Création de sous-processus réutilisables

✅ **Utilisation de Bonita Studio**
- Création et configuration de processus
- Gestion des Call Activities
- Déploiement sur le moteur BPM Engine
- Tests via le Bonita Portal

✅ **Analyse de services web**
- Lecture et compréhension d'un fichier WSDL
- Identification des opérations et messages
- Compréhension du protocole SOAP

### Difficultés rencontrées et solutions

#### 🔴 Problème 1 : Gateway mal configuré
**Symptôme :** Toutes les branches (CB, Chèque, Espèce) s'exécutaient en parallèle

**Cause :** Utilisation d'un AND Gateway au lieu d'un XOR Gateway

**Solution :** Remplacement par un Exclusive Gateway (XOR) avec conditions mutuellement exclusives

---

#### 🔴 Problème 2 : Call Activity non fonctionnelle
**Symptôme :** Le sous-processus "Paiement_Par_Carte" ne se lançait pas

**Cause :** Mauvaise référence de version (fixée à "1.0" au lieu de "Latest")

**Solution :** Configuration de la Call Activity avec version "Latest"

---

#### 🔴 Problème 3 : Validation du format CIN
**Symptôme :** Acceptation de CIN invalides (ex: "12345")

**Cause :** Absence de validation regex sur le champ

**Solution :** Ajout d'une expression régulière `^[A-Z]{1,2}[0-9]{1,6}$`

---

### Points à améliorer

Pour une version avancée du processus, nous pourrions :

1. **Ajouter une base de données** pour stocker les transactions
2. **Intégrer un connecteur bancaire** pour valider réellement les paiements CB
3. **Ajouter des timers** pour gérer les timeouts (ex: 2 minutes pour saisir le code CB)
4. **Créer des rapports** avec Bonita Analytics pour suivre les KPIs
5. **Implémenter une API REST** pour interconnecter avec d'autres systèmes

### Perspectives

Ce TP constitue une excellente introduction aux technologies BPM et BPEL. Les compétences acquises sont directement applicables dans des contextes professionnels pour :

- L'automatisation de processus métiers
- L'orchestration de services web (SOA)
- La gestion de workflows d'entreprise
- L'intégration de systèmes hétérogènes

---

## 📁 Structure des fichiers du projet

```
TP1_BPM_BPEL/
│
├── README.md                          ← Ce fichier
│
├── bonita_projects/
│   ├── Achat_Article-1.0.bos         ← Processus principal
│   └── Paiement_Par_Carte-1.0.bos    ← Sous-processus
│
├── screenshots/
│   ├── 01_bonita_studio.png
│   ├── 02_processus_principal.png
│   ├── 03_sous_processus.png
│   ├── 04_gateway_config.png
│   ├── 05_call_activity.png
│   ├── 06_bonita_portal.png
│   ├── 07_test_cb.png
│   ├── 08_test_cheque.png
│   └── 09_test_espece.png
│
└── documents/
    ├── TP1_PM.pdf                    ← Énoncé du TP
    └── analyse_wsdl.txt              ← Analyse complète du WSDL
```

---

## 🔗 Liens utiles

- **Documentation Bonita :** https://documentation.bonitasoft.com/
- **Spécification BPMN 2.0 :** https://www.omg.org/spec/BPMN/2.0/
- **Tutoriels BPEL :** https://www.oracle.com/technical-resources/articles/soa/bpel-tutorial.html
- **WSDL Reference :** https://www.w3.org/TR/wsdl20/

