# JADE - Intégration des Abstracts du Conseil Constitutionnel

**Projet TER 2026 - Master 1 MIASHS (Informatique et Cognition)**
**Université Grenoble Alpes**

Ce projet s'inscrit dans le cadre du projet de recherche **JADE** (Justice Algorithmique des Élections). Il vise à enrichir la base de données existante en automatisant l'intégration des "abstracts" (résumés juridiques) des décisions du Conseil Constitutionnel.

---

## 👥 Équipe et Rôles

Ce projet est réalisé en binôme avec une répartition des responsabilités techniques :

### **Milhane RABEHI** 
* **Responsabilités :**
    * Audit de la base de données JADE existante et analyse des sources.
    * Modélisation des données (MCD/MLD) pour les tables "Abstracts".
    * Conception de l'architecture SQL et des requêtes d'intégration.
    * Rédaction des spécifications fonctionnelles.

### **Mahdi DJENNAD** 
* **Responsabilités :**
    * Architecture des scripts d'automatisation (ETL).
    * Développement du module de Parsing (Extraction de texte/Regex).
    * Implémentation de la logique d'insertion via l'ORM (Peewee).
    * Gestion du versionnage et de la qualité du code (Git).

**Encadrement :**
* **Mme Caroline BLIGNY** (Ingénieure de recherche, LJK)
* **M. Romain RAMBAUD** (Professeur de droit, CRJ - Porteur du projet)
* **M. Damien PELLIER** (Professeur, UGA - Suiveur TER)

---

## 📋 Contexte et Objectifs

La base JADE croise actuellement les résultats électoraux et les métadonnées des décisions de justice. L'objectif est d'y ajouter le **contenu sémantique** des décisions (les abstracts) pour permettre des analyses plus fines.

**Les étapes clés :**
1.  **Compréhension métier :** Assimiler la structure d'une décision du Conseil Constitutionnel et le déroulement du contentieux électoral.
2.  **Staging :** Charger les fichiers bruts (2023) dans des tables temporaires PostgreSQL pour analyser les cardinalités.
3.  **Modélisation :** Concevoir le modèle de données "Abstracts" et son lien avec la table `decision` existante.
4.  **Développement :** Créer un script Python capable de lire, nettoyer et insérer les abstracts de manière pérenne.

---

## 🛠 Environnement Technique

Le projet repose sur une stack technique définie par l'équipe JADE :

* **Langage :** Python 3.x
* **Base de Données :** PostgreSQL
* **ORM :** Peewee (Interface Python-SQL)
* **Gestionnaire de paquets :** `uv` ou `venv`
* **Versionnage :** Gricad-Gitlab

---

## 📅 Roadmap (Feuille de Route)

### Phase 1 : Analyse (Décembre - Janvier)
- [x] Prise en main du sujet et du Git.
- [x] Analyse des fichiers sources (XML/JSON/TXT) des abstracts.
- [ ] Chargement des données dans des tables brutes (Staging Area).

### Phase 2 : Conception (Janvier - Février)
- [ ] Modélisation relationnelle (Lien Abstract <-> Décision).
- [ ] Validation du schéma SQL cible.

### Phase 3 : Développement (Février - Avril)
- [ ] Script de connexion et lecture des fichiers sources.
- [ ] Développement du parser (Extraction des ID et du texte).
- [ ] Script d'injection en base via Peewee.

### Phase 4 : Finalisation (Mai - Juin)
- [ ] Tests d'intégrité et validation juridique des données.
- [ ] Mise à jour avec les fichiers de données les plus récents.
- [ ] Documentation technique finale sur le Cloud UGA.

---

## 🚀 Installation

**1. Cloner le dépôt :**
```bash
git clone [https://gricad-gitlab.univ-grenoble-alpes.fr/](https://gricad-gitlab.univ-grenoble-alpes.fr/)/jade-abstracts.git
cd jade-abstracts