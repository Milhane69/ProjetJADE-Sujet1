# 📊 Fiche d'Intégration des Données – Projet JADE

**Objectif du document :** Ce document exhaustif détaille l'intégration de l'ensemble des fichiers sources pour enrichir la base de données JADE. Il répertorie les champs importés dans les tables de staging (`raw_*`), les champs conservés dans les tables finales, les champs exclus ainsi que l'architecture technique du pipeline ETL.

---

# 📂 1. Inventaire et Rôle des Fichiers Sources

Le pipeline JADE s'appuie sur la consolidation de quatre fichiers d'extraction :

## 1. `abstrats-2023-01-12.json` (Fichier Principal)

**Contenu :**
Les abstracts juridiques associés aux décisions du Conseil Constitutionnel, avec leurs identifiants, textes et motifs.

**Rôle :**
C'est le fichier central du pipeline. Il constitue la source principale de la table `abstrats` et alimente les analyses textuelles.

---

## 2. `rubriques-2023-01-12.json` (Référentiel Thématique)

**Contenu :**
La hiérarchie des rubriques juridiques.

**Rôle :**
Permet d'associer chaque abstract à une catégorie juridique exploitable.

---

## 3. `extraction table decisionset.xlsx` (Métadonnées des Décisions)

**Contenu :**
Les informations descriptives des décisions du Conseil Constitutionnel.

**Rôle :**
Permet d'enrichir les abstracts avec les métadonnées des décisions.

---

## 4. `type_sous_type.xlsx` (Typologie)

**Contenu :**
Les types et sous-types de décisions.

**Rôle :**
Permet d'ajouter une classification juridique aux décisions.

---

# ✅ 2. Les Champs CONSERVÉS dans les Tables de Staging (`raw_*`)

Les tables de staging conservent les données proches des fichiers sources afin de permettre les contrôles qualité et les transformations.

---

## A. Depuis `abstrats.json` → `raw_abstrats`

| Champ | Rôle |
| :--- | :--- |
| **id** | Identifiant technique |
| **statut_abstrat_id** | Statut de l'abstract |
| **resume_abstrat_id** | Identifiant du résumé |
| **rubrique_id** | Référence vers la rubrique |
| **decision_id** | Référence vers la décision |
| **solution_implicite** | Information juridique complémentaire |
| **resume_abstrat** | Texte brut de l'abstract |
| **renvoi_rubrique_id** | Référence vers une autre rubrique |
| **motifs** | Motifs invoqués (JSONB) |

---

## B. Depuis `rubriques.json` → `raw_rubriques`

| Champ | Rôle |
| :--- | :--- |
| **id** | Clé primaire rubrique |
| **lib** | Libellé juridique |
| **niveau** | Niveau hiérarchique |
| **num_rubrique** | Numéro de rubrique |
| **parent_id** | Rubrique parente |
| **renvoi_rubrique_id** | Renvoi éventuel |
| **ordre** | Ordre d'affichage |

---

## C. Depuis `decisionset.xlsx` → `raw_decisions`

| Champ | Rôle |
| :--- | :--- |
| **id** | Identifiant de la décision |
| **date_decision** | Date de décision |
| **intitule** | Intitulé officiel |
| **reference_officielle** | Référence officielle |
| **referent** | Référent |
| **lien_site_web** | Lien Conseil Constitutionnel |
| **lien_legifrance** | Lien Légifrance |
| **ecli** | Identifiant européen |
| **type_decision_id** | Type de décision |
| **sous_type_decision_id** | Sous-type de décision |
| **departement_id** | Département concerné |
| **demandeurs** | Demandeurs |
| **defendeurs** | Défendeurs |
| **premiere_saisine_le** | Date de première saisine |
| **affaire_greffe_id** | Identifiant de greffe |

---

## D. Depuis `type_sous_type.xlsx` → `raw_types`

| Champ | Rôle |
| :--- | :--- |
| **id** | Identifiant du type |
| **type_decision_code** | Code du type de décision |
| **sous_type_decision_lib** | Libellé du sous-type |

---

# 🗄️ 3. Les Champs CONSERVÉS dans les Tables Finales

Après nettoyage et transformation, seules les informations jugées utiles sont conservées dans les tables finales.

---

## A. Table finale `abstrats`

| Champ | Rôle |
| :--- | :--- |
| **id_abstrat** | Identifiant unique de l'abstract |
| **reference_officielle** | Référence officielle de la décision |
| **rubrique_id** | Référence vers la rubrique |
| **texte_brut** | Texte juridique principal |
| **motifs_invoques** | Motifs invoqués (JSONB) |

---

## B. Table finale `rubriques`

| Champ | Rôle |
| :--- | :--- |
| **id_rubrique** | Identifiant de la rubrique |
| **num_rubrique** | Numéro hiérarchique |
| **lib** | Libellé juridique |
| **niveau** | Niveau dans l'arborescence |
| **parent_id** | Rubrique parente |

---

## C. Table finale `decision`

| Champ | Rôle |
| :--- | :--- |
| **id_decision** | Identifiant de la décision |
| **id_source** | Identifiant provenant de la source |
| **numero** | Référence officielle |
| **date_dec** | Date de décision |
| **ecli** | Identifiant européen |
| **nature** | Nature de la décision |
| **solution** | Solution rendue |
| **url_cc** | Lien Conseil Constitutionnel |
| **departements** | Département(s) concerné(s) |
| **circonscriptions** | Circonscription(s) concernée(s) |
| **type_requete** | Type de requête |
| **annee_saisine** | Année de saisine |

---

# ❌ 4. Les Champs ÉCARTÉS de l'Analyse IA (Nettoyage)

Bien que certaines données soient importées dans les tables de staging, elles ne sont pas conservées dans les tables finales afin de simplifier l'exploitation.

| Fichier Source | Champ | Raison |
| :--- | :--- | :--- |
| `abstrats.json` | **statut_abstrat_id** | Information technique |
| `abstrats.json` | **resume_abstrat_id** | Identifiant interne |
| `abstrats.json` | **decision_id** | Remplacé par la référence officielle |
| `abstrats.json` | **solution_implicite** | Non retenu dans la structure finale |
| `abstrats.json` | **renvoi_rubrique_id** | Non utilisé dans la table finale |
| `decisionset.xlsx` | **intitule** | Non conservé |
| `decisionset.xlsx` | **referent** | Information secondaire |
| `decisionset.xlsx` | **lien_legifrance** | Non retenu |
| `decisionset.xlsx` | **type_decision_id** | Non retenu |
| `decisionset.xlsx` | **sous_type_decision_id** | Non retenu |
| `decisionset.xlsx` | **demandeurs** | Non retenu |
| `decisionset.xlsx` | **defendeurs** | Non retenu |
| `decisionset.xlsx` | **premiere_saisine_le** | Non retenu |
| `decisionset.xlsx` | **affaire_greffe_id** | Non retenu |
| `rubriques.json` | **renvoi_rubrique_id** | Référence documentaire secondaire non utilisée dans les analyses |
| `rubriques.json` | **ordre** | Utilisé uniquement pour l'ordre d'affichage |

---

# ⚙️ 5. Architecture Globale du Pipeline ETL

## A. Phase d'Import Brut (Tables de Staging)

Les données sont importées directement depuis les fichiers sources vers les tables de staging :

- `raw_abstrats`
- `raw_rubriques`
- `raw_decisions`
- `raw_types`

---

## B. Phase de Nettoyage et Transformation

Les données sont ensuite nettoyées, filtrées et transformées afin d'alimenter les tables finales :

- `abstrats`
- `rubriques`
- `decision`

---

# 📌 Résumé

Le pipeline JADE suit une architecture en deux niveaux :

1. **Tables de staging (`raw_*`)**
   - Conservation des données brutes importées.
   - Contrôle qualité.
   - Préparation des transformations.

2. **Tables finales**
   - Données nettoyées.
   - Structure simplifiée.
   - Optimisation des requêtes SQL et des analyses.

La table centrale du projet est désormais **`abstrats`**, enrichie grâce aux tables **`rubriques`** et **`decision`**.