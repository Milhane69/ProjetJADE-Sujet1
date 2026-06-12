# Documentation Technique – Projet JADE

## Justice Algorithmique des Décisions Électorales

**Version : Production + Distribution Scientifique Portable**

---

# 1. Présentation du Projet

JADE (Justice Algorithmique des Décisions Électorales) est une plateforme d'analyse du contentieux électoral du Conseil constitutionnel.

Le projet a pour objectif :

* d'extraire les décisions électorales ;
* d'isoler les contentieux de l'Assemblée nationale (AN) et du Sénat (SEN) ;
* de structurer les abstracts juridiques ;
* de produire une base analytique exploitable ;
* de fournir un tableau de bord interactif destiné aux chercheurs.

L'architecture repose sur :

* un pipeline ETL Python ;
* une base PostgreSQL de production ;
* une couche analytique Streamlit ;
* une version portable destinée à la diffusion scientifique.

---

# 2. Architecture du Projet

```text
Racine du projet (JADE / ABSTRATS)

├── .venv/
├── .env
├── requirements.txt

├── analytics/
│   ├── app.py
│   └── rapport_statistiquesV2.ipynb

├── data/
│   ├── raw/
│   │   ├── abstrats-2023-01-12.json
│   │   ├── extraction table decisionset.xlsx
│   │   ├── rubriques-2023-01-12.json
│   │   └── type sous type.xlsx
│   └── 202605jade.dump

├── docs/

├── logs/
│   └── qualite_donnees.log

├── sql/
│   ├── create_view_abstrats.sql
│   ├── create_view_jade.sql
│   ├── get_staging_abstrats.sql
│   ├── init_database.sql
│   ├── init_table_rejets.sql
│   ├── insert_raw_abstrats.sql
│   ├── insert_raw_decisions.sql
│   ├── insert_raw_rubriques.sql
│   ├── insert_raw_types.sql
│   ├── insert_rejet.sql
│   └── upsert_abstrat.sql

├── src/
│   ├── admin/
│   │   ├── restore_dump.py
│   │   └── export_db.py
│   │
│   ├── config/
│   │   └── settings.py
│   │
│   └── implementation_abstrats/
│       ├── create_view.py
│       ├── data_quality.py
│       ├── import_abstrats.py
│       ├── import_database.py
│       ├── import_decisions.py
│       ├── import_rubriques.py
│       ├── import_types.py
│       ├── init_rejets.py
│       ├── run_pipeline.py
│       └── test_rejet.py

├── tests/
│   └── test_data_quality.py

└── Jade_portable/
    ├── Lancer_JADE.bat
    ├── Lancer_JADE.sh
    ├── app.py
    ├── requirements.txt
    └── data/
        └── jade_export.csv
```

---

# 3. Architecture Fonctionnelle

Le système est organisé en deux environnements distincts.

## A. Environnement de Production

Il comprend :

* les scripts ETL ;
* la base PostgreSQL ;
* les procédures de contrôle qualité ;
* les vues analytiques ;
* les exports.

Répertoires concernés :

```text
src/
sql/
analytics/
```

Cette partie est réservée aux administrateurs du projet.

---

## B. Environnement de Diffusion Scientifique

Le répertoire :

```text
Jade_portable/
```

constitue une version autonome destinée aux chercheurs.

Cette version ne nécessite :

* aucune base PostgreSQL ;
* aucune installation de l'ETL ;
* aucun accès aux données brutes.

Elle repose uniquement sur :

```text
jade_export.csv
+
Streamlit
```

---

# 4. Pipeline de Traitement des Données

Le pipeline suit les étapes suivantes :

```text
Sources brutes
        ↓
Import JSON / Excel
        ↓
Tables RAW PostgreSQL
        ↓
Contrôle qualité
        ↓
Insertion Production
        ↓
Création des vues SQL
        ↓
Export jade_export.csv
        ↓
Dashboard Streamlit
```

---

# 5. Filtrage du Contentieux Électoral

## Objectif

Conserver exclusivement :

* Assemblée nationale (AN)
* Sénat (SEN)

et exclure :

* QPC
* DC
* LP
* LOM
* Référendums
* Contrôle des lois
* Contentieux institutionnels

---

# 6. Étape 1 : Filtre Avant-Garde

## Fichier

```text
src/implementation_abstrats/data_quality.py
```

Avant tout traitement, chaque référence est examinée.

Condition :

```python
if not ref_brute or (
    "AN" not in str(ref_brute).upper()
    and "SEN" not in str(ref_brute).upper()
):
    continue
```

Les dossiers hors périmètre sont immédiatement ignorés.

Ils ne sont :

* ni insérés ;
* ni rejetés ;
* ni journalisés.

Ils disparaissent du pipeline.

---

# 7. Normalisation des Références

La méthode :

```python
clean_reference()
```

uniformise les références.

Exemple :

```text
Décision n° 73-690 AN du 01 juin 1973
```

devient :

```text
73-690
```

Les opérations réalisées sont :

* suppression des espaces ;
* suppression des espaces insécables ;
* suppression de « Décision n° » ;
* suppression des dates ;
* extraction du numéro officiel.

Cette normalisation permet la jointure avec les décisions.

---

# 8. Contrôle des Rubriques

Le système insère automatiquement :

### Cas 1

Toutes les rubriques :

```text
8.*
```

correspondant au contentieux électoral.

---

### Cas 2

Les rubriques associées à une décision :

```text
AN
ou
SEN
```

même si leur code n'appartient pas à la famille 8.

Cette stratégie évite l'exclusion de dossiers électoraux valides.

---

# 9. Contrôle des Abstracts

Chaque abstract est ensuite évalué.

## Rejet n°1

Texte vide.

```text
Longueur < 10 caractères
```

---

## Rejet n°2

Rubrique absente.

```text
Rubrique exclue ou introuvable
```

---

## Rejet n°3

Référence orpheline.

La décision n'existe pas dans la table des décisions.

---

## Validation

L'abstract est inséré dans la base de production.

---

# 10. Construction des Vues SQL

## Fichiers

```text
sql/create_view_jade.sql
sql/create_view_abstrats.sql
```

Les vues analytiques réalisent :

* les jointures ;
* les enrichissements ;
* les optimisations.

---

## Vue électorale

```sql
WHERE dec.nature IN ('AN','SEN')
```

---

## Vue hors électorale

```sql
WHERE dec.nature NOT IN ('AN','SEN')
   OR dec.nature IS NULL
```

---

# 11. Export Scientifique

## Script

```text
src/admin/export_db.py
```

Ce script exporte :

```text
v_abstracts_jade
```

vers :

```text
Jade_portable/data/jade_export.csv
```

---

# 12. Structure du Fichier CSV

```csv
reference_officielle,
date_decision,
annee,
code_nature,
theme_juridique,
num_rubrique,
rubrique_lib,
solution,
ecart_voix,
texte_brut,
id_abstract
```

Exemple :

```csv
73-690,1973-06-01,1973,AN,Propagande,8.3,Tracts,Annulation,24,"Considérant que...",1

73-690,1973-06-01,1973,AN,Diffamation,8.4,Campagne,Annulation,24,"Considérant que...",2
```

---

# 13. Tableau de Bord JADE

## Application

```text
analytics/app.py
```

Version production.

---

## Application Portable

```text
Jade_portable/app.py
```

Version destinée aux chercheurs.

Fonctionnalités :

* KPIs ;
* recherche plein texte ;
* filtres interactifs ;
* statistiques descriptives ;
* graphiques temporels ;
* visualisations Plotly ;
* audit qualité ;
* analyse exploratoire ;
* outils d'aide à la décision.

---

# 14. Distribution aux Chercheurs

La diffusion scientifique repose sur le dossier :

```text
Jade_portable/
```

---

## Sous Windows

L'utilisateur exécute :

```text
Lancer_JADE.bat
```

---

## Sous Linux / macOS

L'utilisateur exécute :

```bash
./Lancer_JADE.sh
```

---

## Fonctionnement

```text
Chercheur
     ↓
Lancer_JADE.bat ou Lancer_JADE.sh
     ↓
Démarrage Streamlit
     ↓
Ouverture automatique du navigateur
     ↓
Consultation du tableau de bord JADE
```

---

# 15. Avantages de l'Architecture

## Simplicité

Aucune installation PostgreSQL.

## Sécurité

Les données brutes restent dans l'environnement de production.

## Reproductibilité

Tous les chercheurs utilisent le même export figé.

## Portabilité

Compatible Windows, Linux et macOS.

## Pérennité

Aucune dépendance à un serveur externe.

## Souveraineté des données

Le projet reste entièrement maîtrisé par l'équipe de recherche.

---

# Conclusion

JADE repose sur une architecture en deux niveaux :

1. une plateforme de production assurant l'ingestion, le nettoyage, la validation et l'export des données électorales ;

2. une plateforme portable permettant aux chercheurs d'explorer les décisions électorales du Conseil constitutionnel via une interface Streamlit autonome alimentée par un export figé de la base analytique.

Cette organisation garantit à la fois la qualité scientifique des données, la reproductibilité des analyses et la maîtrise complète des ressources par l'équipe de recherche.
