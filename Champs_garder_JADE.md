# 📊 Fiche d'Intégration des Données – Projet JADE

**Objectif du document :** Ce document exhaustif détaille l'intégration de l'ensemble des fichiers sources pour enrichir la table `abstracts` avec : le texte juridique, les rubriques thématiques, les métadonnées des décisions, et les types juridiques associés. Il répertorie la totalité des champs (conservés pour le staging, conservés pour l'IA, et exclus) ainsi que l'architecture technique du pipeline ETL.

---

## 📂 1. Inventaire et Rôle des Fichiers Sources

Le pipeline JADE s'appuie sur la consolidation de quatre fichiers d'extraction :

1. **`abstrats-2023-01-12.json` (Fichier Principal)**
   * **Contenu :** Les abstracts juridiques associés aux décisions du CC, avec identifiants, textes et motifs.
   * **Rôle :** C'est le fichier central du pipeline (base de la table `abstracts`). Il alimentera l'analyse textuelle, la similarité sémantique, l'extraction de mots-clés, le clustering et l'analyse LLM.

2. **`rubriques-2023-01-12.json` (Référentiel Thématique)**
   * **Contenu :** La hiérarchie thématique juridique (ex: 80724 → Pouvoir constituant).
   * **Rôle :** Permet d'interpréter le `RubriqueId` via une jointure (`abstracts.RubriqueId = rubriques.Id`) pour le transformer en catégorie juridique exploitable.

3. **`extraction table decisionset.xlsx` (Métadonnées)**
   * **Contenu :** Les métadonnées complètes des décisions (dates, intitulés, liens).
   * **Rôle :** Fichier de mapping essentiel pour relier le `DecisionId` de l'abstract au numéro officiel de la décision.

4. **`type_sous_type.xlsx` (Typologie)**
   * **Contenu :** Traduction des identifiants numériques de types de décisions.
   * **Rôle :** Donne du sens au `TypeDecisionId` (ex: 3 → QPC). Permet le filtrage, les analyses statistiques et l'enrichissement sémantique.

---

## ✅ 2. Les Champs CONSERVÉS (Par Fichier)

Cette section liste tous les champs importés dans la base de données (Staging), dont les plus importants formeront le socle de l'analyse IA.

### A. Depuis `abstrats.json`
| Champ | Rôle | Exemple issu du corpus |
| :--- | :--- | :--- |
| **Id** | Identifiant technique | *294725* |
| **ResumeAbstratId** | Identifiant du résumé | *250247* |
| **RubriqueId** | Clé de jointure vers la thématique | *80726* |
| **DecisionId** | Clé de jointure vers la décision | *69512* |
| **SolutionImplicite** | Info juridique complémentaire | *Null* ou texte |
| **ResumeAbstrat** | Texte principal à analyser par l'IA | *"Sous réserve, d'une part, des limitations..."* |
| **RenvoiRubriqueId** | Lien vers rubrique associée | *Null* ou ID |
| **motifs** | Points de droits invoqués | `["19", "20"]` |
| **StatutAbstratId** | Filtre de publication (Staging) | *3* |

### B. Depuis `rubriques.json`
| Champ | Rôle | Exemple issu du corpus |
| :--- | :--- | :--- |
| **Id** | Clé primaire rubrique | *80726* |
| **Lib** | Nom de la rubrique juridique | *"Étendue du pouvoir de révision"* |
| **Niveau** | Profondeur hiérarchique | *5* |
| **NumRubrique** | Position dans l'arbre | *"1.1.1.1.1"* |
| **ParentId** | Rubrique parente | *80725* |
| **RenvoiRubriqueId**| Relation transversale | *Null* ou ID |
| **Ordre** | Ordre d'affichage (Staging) | *1* |

### C. Depuis `decisionset.xlsx` (Champs Critiques & Secondaires)
| Champ | Rôle | Exemple issu du corpus |
| :--- | :--- | :--- |
| **Id** | Identifiant de la décision | *66644* |
| **Date** | Date de la décision | `2010-05-12` |
| **Intitule** | Titre de la décision | *"Loi relative à l'ouverture à la concurrence..."* |
| **ReferenceOfficielle**| Numéro officiel | *"2010-605 DC"* |
| **LienSiteWeb** | URL officielle | *2010/2010605DC.htm* |
| **LienLegifrance** | URL Légifrance | *CSCL1012832S* |
| **ECLI** | Identifiant européen | *ECLI:FR:CC:2010:2010.605.DC* |
| **TypeDecisionId** | Type global | *2* |
| **SousTypeDecisionId** | Sous-type spécifique | *7* |
| **Champs Secondaires** | Pour analyses avancées | *DepartementId, Demandeurs, Defendeurs, PremiereSaisineLe, AffaireGreffeId* |

### D. Depuis `type_sous_type.xlsx`
| Champ | Rôle | Exemple issu du corpus |
| :--- | :--- | :--- |
| **TypeDecisionSet Code** | Code global | *QPC, DC, LP* |
| **SousTypeDecisionSet Lib**| Nom du type juridique | *"Question prioritaire de constitutionnalité"* |

---

## ❌ 3. Les Champs ÉCARTÉS de l'Analyse IA (Nettoyage)

Bien que certaines données soient importées en "Staging", elles seront exclues de la table finale `abstracts` fournie au modèle d'Intelligence Artificielle afin de réduire le bruit.

| Fichier Source | Champ | Raison de l'exclusion pour l'IA |
| :--- | :--- | :--- |
| `abstrats.json` | **StatutAbstratId** | Indique le cycle de publication (brouillon, validation). JADE ne travaille que sur le corpus publié. |
| `rubriques.json` | **Ordre, Niveau, ParentId** | Données de mise en page web (arborescence du site). L'IA n'a besoin que du libellé direct (`Lib`), pas de sa position dans un menu. |
| `decisionset.xlsx` | **EstAnonymisee** | Information administrative (masquage des noms pour la vie privée). N'impacte pas le raisonnement juridique et les motifs analysés. |
| `decisionset.xlsx` | **TitreCommercial** | Titre raccourci pour la presse/communication, souvent vide et moins rigoureux que l'intitulé officiel. |
| `decisionset.xlsx` | **DatePublicationJO** | Pour l'étude chronologique, c'est la `Date` de la décision qui fait foi juridiquement, pas le délai d'impression au Journal Officiel. |

---

## ⚙️ 4. Architecture Globale du Pipeline ETL

### A. Le Problème Technique et sa Solution
* **Problème :** Le champ `DecisionId` (dans les abstracts) ne correspond pas directement au format `decision.numero` (ex: 2010-605 DC) attendu dans PostgreSQL.
* **Solution :** Utiliser `decisionset.xlsx` comme **table de correspondance** (Pivot) pour faire la traduction via une jointure.

### B. Architecture SQL Recommandée
L'intégration des données doit se faire impérativement en deux temps pour garantir l'intégrité :

1.  **Phase d'Import Brut (Tables Staging) :**
    * `abstracts_raw`
    * `rubriques_raw`
    * `decisions_raw`
    * `types_raw`

2.  **Phase de Nettoyage et Jointure (Tables Finales) :**
    * `rubriques`
    * `decisions`
    * `types_decision`
    * **`abstracts` (La table cible)**

**🎯 Résultat Attendu :**
La construction d'une table finale enrichie (`abstracts`) contenant sur une seule et même ligne :
* Le texte juridique brut.
* Le thème juridique en clair.
* La décision associée (référence et date).
* Le type de procédure juridique.
