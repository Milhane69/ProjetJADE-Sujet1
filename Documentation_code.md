# PROJET JADE
## Documentation Technique – Version 1

---

# 1. Présentation du projet

Le projet JADE est un pipeline ETL juridique développé en Python et PostgreSQL.

L’objectif principal du projet est de :

- importer des données juridiques provenant de fichiers JSON et Excel ;
- nettoyer et transformer les données ;
- enrichir les données via des jointures SQL ;
- construire une base exploitable pour l’analyse juridique ;
- préparer les données pour de futures analyses NLP / IA.

Le projet se concentre principalement sur les décisions électorales du Conseil Constitutionnel.

---

# 2. Architecture générale du projet

Le pipeline suit l’architecture suivante :

```text
Fichiers sources
(JSON / Excel)
        ↓
Scripts Python ETL
        ↓
Tables staging (_raw)
        ↓
Nettoyage et validation qualité
        ↓
Table finale abstracts
        ↓
Vue SQL optimisée
        ↓
Analyse juridique / NLP / IA
```

---

# 3. Environnement utilisé

## 3.1 Système et outils

- Windows
- Visual Studio Code
- PostgreSQL 17 / 18
- pgAdmin 4
- Python 3.12
- Git / GitLab

---

## 3.2 Bibliothèques Python utilisées

| Bibliothèque | Rôle |
|---|---|
| psycopg2 | Connexion PostgreSQL |
| pandas | Lecture des fichiers Excel |
| json | Lecture des fichiers JSON |
| dotenv | Chargement sécurisé des variables d’environnement |
| logging | Gestion des logs d’erreurs |
| subprocess | Exécution automatique des scripts |
| re | Nettoyage via expressions régulières |
| BeautifulSoup (bs4) | Parsing XML / HTML |

---

# 4. Sources de données

## 4.1 Fichiers JSON

### abstrats-2023-01-12.json

Contient les abstracts juridiques.

Champs principaux :

- Id
- ResumeAbstrat
- DecisionId
- RubriqueId
- motifs

---

### rubriques-2023-01-12.json

Contient la hiérarchie des rubriques juridiques.

Permet de transformer un identifiant technique en thème juridique compréhensible.

---

## 4.2 Fichiers Excel

### extraction table decisionset.xlsx

Contient les métadonnées des décisions :

- date_decision
- reference_officielle
- ecli
- type_decision_id
- sous_type_decision_id

---

### type sous type.xlsx

Contient les types et sous-types de décisions.

Exemple :

- AN
- SEN
- QPC
- DC

---

# 5. Variables d’environnement

Le projet utilise un fichier `.env` pour sécuriser les informations sensibles.

## Exemple :

```env
DB_NAME=JADE
DB_USER=postgres
DB_PASSWORD=mot_de_passe
DB_HOST=localhost
DB_PORT=5432
```

---

## Fichier `.env_template`

Le fichier `.env_template` sert de modèle pour les autres développeurs.

Il ne contient jamais le vrai mot de passe.

---

# 6. Gestion Git et sécurité

## 6.1 `.gitignore`

Le fichier `.gitignore` empêche l’envoi des fichiers sensibles ou inutiles.

Exemples :

```text
.venv/
.env
logs/
__pycache__/
.vscode/
```

---

## 6.2 Sécurité

Les informations sensibles ne doivent jamais être poussées sur GitLab :

- mot de passe PostgreSQL ;
- dump de production ;
- environnement virtuel ;
- logs locaux.

---

# 7. Tables PostgreSQL

# 7.1 Tables de staging

Les tables staging servent à stocker les données brutes avant transformation.

---

## 7.1.1 abstracts_raw

Contient les abstracts juridiques.

Colonnes principales :

- id
- decision_id
- rubrique_id
- resume_abstrat
- motifs

Le champ `motifs` est stocké en JSONB.

---

## 7.1.2 rubriques_raw

Contient les rubriques juridiques.

Permet d’obtenir les thèmes juridiques lisibles.

---

## 7.1.3 decisions_raw

Contient les métadonnées des décisions.

Apporte :

- date_decision
- reference_officielle
- ecli
- type_decision_id
- sous_type_decision_id

---

## 7.1.4 types_raw

Contient les types et sous-types de décisions.

Permet de transformer un identifiant technique en libellé compréhensible.

---

# 7.2 Table finale : abstracts

La table `abstracts` est la table finale enrichie.

Elle est construite après les jointures entre :

- abstracts_raw
- rubriques_raw
- decisions_raw
- types_raw

Colonnes principales :

- id_abstract
- reference_officielle
- date_decision
- theme_juridique
- texte_brut
- motifs_invoques
- type_recours

Cette table est utilisée pour :

- l’analyse juridique ;
- les recherches SQL ;
- le NLP ;
- les futurs traitements IA.

---

# 8. Scripts SQL

# 8.1 01_init_database.sql

Ce script :

- crée les tables staging ;
- crée la table finale ;
- crée les index PostgreSQL.

Index créés :

- idx_abstracts_reference
- idx_abstracts_theme
- idx_abstracts_type

Un index GIN peut être activé pour la recherche plein texte.

---

# 8.2 02_Init_rejet.sql

Crée la table `abstracts_rejets`.

Cette table stocke les données invalides rejetées pendant le pipeline.

---

# 8.3 03_init_test_qualité.sql

Contient des requêtes SQL permettant de tester :

- les jointures ;
- la qualité des données ;
- la cohérence des références.

---

# 8.4 04_Finition_JADE.sql

Crée une vue SQL intelligente :

```sql
v_abstracts_jade
```

La vue permet :

- d’accélérer les recherches ;
- de simplifier les requêtes ;
- de préparer les analyses futures.

---

# 8.5 requete_sans_la_vue.sql

Contient des exemples de jointures SQL sans utiliser la vue.

---

# 8.6 type_de_requete.sql

Contient des exemples de requêtes :

- recherche par thème ;
- tri par date ;
- affichage complet du corpus.

---

# 9. Scripts Python ETL

# 9.1 connexion_bdd.py

Petit script de test.

Permet de vérifier que PostgreSQL est accessible depuis Python.

---

# 9.2 import_database.py

Script principal d’initialisation.

Rôle :

- supprimer les anciennes tables ;
- recréer les tables ;
- recréer les index.

Le script utilise une grande variable SQL nommée :

```python
SQL_SCRIPT
```

qui contient toute la structure SQL.

---

# 9.3 import_abstracts.py

Importe le fichier JSON des abstracts.

Fonctionnement :

```python
json.load(f)
```

convertit le JSON en dictionnaire Python.

Ensuite :

```python
for key, value in data.items()
```

parcourt chaque abstract.

Le champ `motifs` est converti en JSONB PostgreSQL grâce à :

```python
json.dumps()
```

---

# 9.4 import_rubriques.py

Importe les rubriques juridiques depuis le JSON.

Le fonctionnement est similaire à `import_abstracts.py`.

---

# 9.5 import_decisions.py

Importe les décisions depuis Excel.

Le script utilise :

```python
pd.read_excel()
```

pour lire le fichier.

Puis :

```python
df.iterrows()
```

pour parcourir chaque ligne.

Le script nettoie également les tailles des champs VARCHAR.

---

# 9.6 import_types.py

Importe les types et sous-types de décisions.

Ce script alimente la table `types_raw`.

---

# 9.7 init_rejets.py

Crée automatiquement la table :

```sql
abstracts_rejets
```

Le script contient également des protections contre :

- l’absence du fichier `.env` ;
- un mot de passe vide.

---

# 9.8 data_quality.py

Script central du pipeline.

Il réalise :

- les jointures ;
- le nettoyage ;
- la validation qualité ;
- la gestion des rejets ;
- l’insertion dans la table finale.

---

## Règles qualité appliquées

### Référence obligatoire

Une décision sans référence officielle est rejetée.

---

### Date obligatoire

Une décision sans date est rejetée.

---

### Longueur minimale

Un abstract inférieur à 10 caractères est rejeté.

---

## Nettoyage des références

La fonction :

```python
clean_reference()
```

utilise des expressions régulières pour normaliser les références.

Exemple :

```text
Décision n° 2013-4764 AN
→ 2013-4764AN
```

---

## Correction d’encodage

La fonction :

```python
fix_encoding()
```

corrige certains problèmes d’encodage UTF-8.

---

## Gestion des logs

Les erreurs sont enregistrées dans :

```text
logs/qualite_donnees.log
```

Exemple :

```text
La date de décision est manquante.
```

---

## Filtre électoral

Le pipeline conserve uniquement les décisions contenant :

- AN
- SEN

Ce filtre permet de cibler uniquement les contentieux électoraux.

---

# 9.9 create_view.py

Crée la vue SQL :

```sql
vue_abstracts_jade
```

Cette vue simplifie les analyses SQL.

Elle réalise des jointures avancées entre :

- abstracts
- decision

---

# 9.10 run_pipeline.py

Script d’automatisation global.

Il exécute automatiquement :

1. init_rejets.py
2. import_database.py
3. import_types.py
4. import_rubriques.py
5. import_decisions.py
6. import_abstracts.py
7. data_quality.py
8. restauration du dump
9. create_view.py

Le script utilise :

```python
subprocess.run()
```

pour lancer les scripts Python automatiquement.

---

# 9.11 test_rejet.py

Script de test qualité.

Injecte volontairement une mauvaise donnée :

- date NULL ;
- texte trop court.

Permet de vérifier que le système de rejet fonctionne correctement.

---

# 9.12 parser_prototype.py

Prototype expérimental utilisant BeautifulSoup.

Objectif :

- lire du XML/HTML ;
- extraire des informations juridiques ;
- détecter des mots-clés via REGEX.

Le script détecte par exemple :

- fraude ;
- sincérité ;
- manoeuvre.

Ce prototype prépare une future extraction automatique depuis des décisions XML.

---

# 10. Logs et gestion des erreurs

Le projet utilise le module Python :

```python
logging
```

Les erreurs qualité sont stockées dans :

```text
logs/qualite_donnees.log
```

Les données invalides sont stockées dans :

```sql
abstracts_rejets
```

Cela permet :

- d’éviter la perte des données ;
- de déboguer les anomalies ;
- d’améliorer progressivement le pipeline.

---

# 11. Vue SQL finale

La vue :

```sql
vue_abstracts_jade
```

permet :

- les recherches rapides ;
- les tris par date ;
- les analyses juridiques ;
- les futures analyses IA.

---

# 12. Exemple de requêtes SQL

## Afficher tout le corpus

```sql
SELECT * FROM v_abstracts_jade;
```

---

## Recherche par thème

```sql
SELECT *
FROM v_abstracts_jade
WHERE theme_juridique LIKE '%Fraude%';
```

---

## Tri chronologique

```sql
SELECT *
FROM v_abstracts_jade
ORDER BY date_dec DESC;
```

---

# 13. Pipeline complet

## Ordre d’exécution recommandé

```text
1. import_database.py
2. import_types.py
3. import_rubriques.py
4. import_decisions.py
5. import_abstracts.py
6. data_quality.py
7. create_view.py
```

Ou directement :

```bash
python src/etl/run_pipeline.py
```

---

# 14. Objectifs futurs

Le projet pourra évoluer vers :

- NLP juridique ;
- recherche sémantique ;
- classification automatique ;
- détection de fraude ;
- analyse temporelle ;
- intégration LLM ;
- moteur de recherche juridique intelligent.

---

# 15. Méthodes importantes utilisées dans le projet

Cette partie présente uniquement les méthodes importantes utilisées dans les fichiers du projet JADE. Pour chaque méthode, on indique son rôle et la fonction utilisée.

---

# 15.1 Méthodes importantes dans `import_abstracts.py`

## Charger le fichier JSON des abstracts

Convertit le fichier JSON en dictionnaire Python.

```python
json.load(f)
```

---

## Parcourir les abstracts du fichier JSON

Permet de parcourir chaque élément du dictionnaire JSON.

```python
for key, value in data.items():
```

---

## Récupérer une valeur JSON sans provoquer d’erreur

Permet de récupérer une valeur même si la clé n’existe pas.

```python
value.get("Id")
```

---

## Convertir les motifs en JSONB PostgreSQL

Transforme une liste Python en format JSON compatible avec PostgreSQL.

```python
json.dumps(value.get("motifs", []))
```

---

## Exécuter une insertion SQL

Permet d’insérer les abstracts dans la table `abstracts_raw`.

```python
cur.execute(...)
```

---

## Valider les insertions

Sauvegarde définitivement les données dans PostgreSQL.

```python
conn.commit()
```

---

## Fermer le curseur et la connexion

Ferme proprement la connexion à PostgreSQL.

```python
cur.close()
conn.close()
```

---

# 15.2 Méthodes importantes dans `import_rubriques.py`

## Charger le fichier JSON des rubriques

Convertit le fichier JSON en dictionnaire Python.

```python
json.load(f)
```

---

## Parcourir les rubriques

Permet de parcourir chaque rubrique du fichier JSON.

```python
for key, value in data.items():
```

---

## Récupérer les champs d’une rubrique

Récupère les valeurs comme `Id`, `Lib`, `Niveau`, `ParentId`.

```python
value.get("Lib")
```

---

## Insérer les rubriques dans PostgreSQL

Exécute l’insertion dans la table `rubriques_raw`.

```python
cur.execute(...)
```

---

## Valider les insertions

Sauvegarde définitivement les rubriques importées.

```python
conn.commit()
```

---

# 15.3 Méthodes importantes dans `import_decisions.py`

## Lire le fichier Excel des décisions

Charge le fichier Excel dans un DataFrame pandas.

```python
pd.read_excel("./data/raw/extraction table decisionset.xlsx")
```

---

## Parcourir les lignes Excel

Permet de parcourir chaque décision du fichier Excel.

```python
for _, row in df.iterrows():
```

---

## Détecter les cellules vides

Permet de remplacer les valeurs vides Excel par `NULL` dans PostgreSQL.

```python
pd.isna(row.get("Date"))
```

---

## Convertir les identifiants en entiers

Transforme les identifiants Excel en entiers PostgreSQL.

```python
int(row.get("Id"))
```

---

## Limiter la taille des champs texte

Évite les erreurs avec les colonnes `VARCHAR(50)` ou `VARCHAR(100)`.

```python
str(row.get("ReferenceOfficelle"))[:50]
```

---

## Insérer les décisions dans PostgreSQL

Insère les données dans la table `decisions_raw`.

```python
cur.execute(...)
```

---

# 15.4 Méthodes importantes dans `import_types.py`

## Lire le fichier Excel des types

Charge le fichier `type sous type.xlsx`.

```python
pd.read_excel("./data/raw/type sous type.xlsx")
```

---

## Parcourir les lignes du fichier Excel

Permet de lire chaque type et sous-type de décision.

```python
for _, row in df.iterrows():
```

---

## Gérer les cellules vides

Remplace les cellules vides par `NULL`.

```python
None if pd.isna(row.get("SousTypeDecisionSet Id")) else int(row.get("SousTypeDecisionSet Id"))
```

---

## Insérer les types dans PostgreSQL

Insère les types dans la table `types_raw`.

```python
cur.execute(...)
```

---

# 15.5 Méthodes importantes dans `import_database.py`

## Charger les variables du fichier `.env`

Récupère automatiquement les informations de connexion PostgreSQL.

```python
load_dotenv()
```

---

## Récupérer les variables d’environnement

Permet de récupérer `DB_NAME`, `DB_USER`, `DB_PASSWORD`, etc.

```python
os.getenv("DB_NAME")
```

---

## Se connecter à PostgreSQL

Ouvre une connexion avec la base de données.

```python
psycopg2.connect(**DB_CONFIG)
```

---

## Exécuter le script SQL complet

Supprime et recrée les tables du projet.

```python
cur.execute(SQL_SCRIPT)
```

---

## Annuler en cas d’erreur

Annule les modifications si une erreur arrive pendant l’initialisation.

```python
conn.rollback()
```

---

# 15.6 Méthodes importantes dans `data_quality.py`

## Configurer les logs

Définit le fichier où les erreurs seront enregistrées.

```python
logging.basicConfig(...)
```

---

## Créer un logger

Permet d’écrire des erreurs dans le fichier de logs.

```python
logger = logging.getLogger("ETL_QUALITY")
```

---

## Convertir une date en JSON

Permet à `json.dumps()` de gérer les objets `date` et `datetime`.

```python
def json_serial(obj):
```

---

## Nettoyer une référence officielle

Transforme une référence longue en référence normalisée.

```python
def clean_reference(ref):
```

Exemple :

```text
Décision n° 2013-4764 AN
→ 2013-4764AN
```

---

## Supprimer les mots parasites

Supprime par exemple `DÉCISION N°` dans une référence.

```python
re.sub(r'DÉCISION(?:N°|NO)?', '', ref_str)
```

---

## Chercher un format de référence

Recherche une référence comme `2013-4764AN`.

```python
re.search(r'[0-9]{2,4}-[0-9/]+[A-Z]*', ref_str)
```

---

## Corriger certains problèmes d’encodage

Corrige des textes mal encodés comme `Ã©`.

```python
def fix_encoding(text):
```

---

## Récupérer les lignes sous forme de dictionnaire

Permet d’accéder aux colonnes par leur nom.

```python
psycopg2.extras.RealDictCursor
```

---

## Faire les jointures entre les tables staging

Relie `abstracts_raw`, `decisions_raw`, `rubriques_raw` et `types_raw`.

```sql
LEFT JOIN decisions_raw d ON a.decision_id = d.id
LEFT JOIN rubriques_raw r ON a.rubrique_id = r.id
LEFT JOIN types_raw t ON d.sous_type_decision_id = t.id
```

---

## Filtrer uniquement les décisions électorales

Conserve seulement les références contenant `AN` ou `SEN`.

```python
if "AN" not in ref_propre and "SEN" not in ref_propre:
    continue
```

---

## Insérer ou mettre à jour les abstracts valides

Évite les doublons avec `ON CONFLICT`.

```sql
ON CONFLICT (id_abstract) DO UPDATE SET
```

---

## Stocker les lignes invalides dans la table des rejets

Insère les erreurs dans `abstracts_rejets`.

```python
cur.execute(...)
```

---

## Écrire une erreur dans le fichier de logs

Ajoute une ligne dans `qualite_donnees.log`.

```python
error_logger.error(...)
```

---

# 15.7 Méthodes importantes dans `create_view.py`

## Charger explicitement le fichier `.env`

Charge le vrai fichier `.env` situé à la racine du projet.

```python
load_dotenv(dotenv_path=chemin_env)
```

---

## Supprimer les espaces invisibles

Évite des erreurs de connexion sous Windows.

```python
str(os.getenv("DB_NAME")).strip()
```

---

## Créer ou remplacer la vue SQL

Crée la vue finale `vue_abstracts_jade`.

```sql
CREATE OR REPLACE VIEW public.vue_abstracts_jade AS
```

---

## Nettoyer les références dans la jointure

Supprime les espaces classiques et les espaces insécables.

```sql
REPLACE(REPLACE(UPPER(dec.numero), ' ', ''), CHR(160), '')
```

---

## Comparer numéro + nature

Permet de faire correspondre des références comme `2013-4764AN`.

```sql
UPPER(dec.numero || dec.nature)
```

---

## Créer un index

Accélère les recherches sur `reference_officielle`.

```sql
CREATE INDEX IF NOT EXISTS idx_abstracts_ref
ON public.abstracts(reference_officielle);
```

---

# 15.8 Méthodes importantes dans `init_rejets.py`

## Construire le chemin du fichier `.env`

Permet de retrouver le fichier `.env` à la racine du projet.

```python
os.path.join(BASE_DIR, ".env")
```

---

## Vérifier si `.env` existe

Évite que le script plante sans explication.

```python
os.path.exists(chemin_env)
```

---

## Arrêter le script en cas d’erreur critique

Stoppe l’exécution si `.env` est introuvable ou incomplet.

```python
sys.exit(1)
```

---

## Vérifier si le mot de passe existe

Contrôle que `DB_PASSWORD` n’est pas vide.

```python
db_pass = os.getenv("DB_PASSWORD")
```

---

## Créer la table des rejets

Crée la table `abstracts_rejets` si elle n’existe pas.

```sql
CREATE TABLE IF NOT EXISTS public.abstracts_rejets
```

---

# 15.9 Méthodes importantes dans `run_pipeline.py`

## Définir la racine du projet

Calcule le chemin absolu du projet.

```python
BASE_DIR = os.path.abspath(os.path.join(os.path.dirname(__file__), "..", ".."))
```

---

## Forcer l’affichage UTF-8 sous Windows

Évite les problèmes d’accents dans le terminal.

```python
sys.stdout = io.TextIOWrapper(sys.stdout.buffer, encoding='utf-8')
```

---

## Définir le Python de l’environnement virtuel

Permet d’exécuter les scripts avec le bon Python.

```python
PYTHON_EXE = os.path.join(BASE_DIR, ".venv", "Scripts", "python.exe")
```

---

## Définir l’ordre des scripts ETL

Liste les scripts à exécuter dans l’ordre.

```python
python_tasks = [...]
```

---

## Exécuter un script Python automatiquement

Lance chaque script du pipeline.

```python
subprocess.run([PYTHON_EXE, full_path], capture_output=False)
```

---

## Arrêter le pipeline si une tâche échoue

Empêche de continuer si un script a échoué.

```python
if result.returncode != 0:
    sys.exit(1)
```

---

## Restaurer le dump PostgreSQL

Utilise `pg_restore.exe` pour restaurer les données historiques.

```python
subprocess.run(cmd, env=env, check=True)
```

---

## Passer le mot de passe à PostgreSQL

Évite de taper le mot de passe manuellement.

```python
env["PGPASSWORD"] = DB_CONFIG["password"]
```

---

# 15.10 Méthodes importantes dans `test_rejet.py`

## Injecter une fausse décision

Ajoute une décision volontairement incomplète pour tester le rejet.

```sql
INSERT INTO decisions_raw (...)
```

---

## Injecter un faux abstract

Ajoute un abstract volontairement trop court.

```sql
INSERT INTO abstracts_raw (...)
```

---

## Éviter les doublons pendant le test

Ignore l’insertion si l’identifiant existe déjà.

```sql
ON CONFLICT (id) DO NOTHING
```

---

# 15.11 Méthodes importantes dans `connexion_bdd.py`

## Tester la connexion PostgreSQL

Vérifie que Python arrive à se connecter à la base.

```python
psycopg2.connect(...)
```

---

## Afficher un message de succès

Confirme que la connexion fonctionne.

```python
print("Connexion à PostgreSQL réussie !")
```

---

## Gérer les erreurs de connexion

Affiche l’erreur si la connexion échoue.

```python
except Exception as e:
```

---

# 15.12 Méthodes importantes dans `parser_prototype.py`

## Parser un document XML

Transforme le XML en objet manipulable avec Python.

```python
BeautifulSoup(xml_content, "xml")
```

---

## Trouver une balise XML

Recherche une balise précise comme `numero` ou `abstract`.

```python
soup.find("numero")
```

---

## Extraire le texte d’une balise

Récupère le contenu textuel d’une balise XML.

```python
soup.find("numero").text
```

---

## Détecter des mots-clés juridiques

Cherche des mots comme `manoeuvre`, `fraude`, `sincérité`.

```python
re.search(r"manœuvre|fraude|sincérité", texte_abstract, re.IGNORECASE)
```

---

## Retourner les données extraites

Retourne un dictionnaire avec le numéro, le texte et les tags.

```python
return {
    "numero_decision": numero,
    "texte": texte_abstract,
    "tags": mots_cles
}
```

---

