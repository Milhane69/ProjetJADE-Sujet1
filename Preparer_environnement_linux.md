# PROJET JADE

## Documentation Technique — Version Linux

---

# 1. Environnement utilisé

Cette documentation a été préparée pour exécuter le projet JADE sous Linux.

## Logiciels nécessaires

- Python 3
- PostgreSQL
- pgAdmin4
- VS Code
- Git

---

# 2. Installation des outils nécessaires

## Mise à jour du système

```bash
sudo apt update && sudo apt upgrade -y
```

## Installation de Python et outils de compilation

```bash
sudo apt install python3-pip python3-venv python3-dev build-essential -y
```

## Installation PostgreSQL

```bash
sudo apt install postgresql postgresql-contrib libpq-dev -y
```

---

# 3. Installation de pgAdmin4

```bash
curl -fsS https://www.pgadmin.org/static/packages_pgadmin_org.pub | sudo gpg --dearmor -o /usr/share/keyrings/packages-pgadmin-org.gpg
```

Ensuite :

```bash
sudo rm /etc/apt/sources.list.d/pgadmin4.list
```

Puis :

```bash
echo "deb [signed-by=/usr/share/keyrings/packages-pgadmin-org.gpg] https://ftp.postgresql.org/pub/pgadmin/pgadmin4/apt/$(lsb_release -cs) pgadmin4 main" | sudo tee /etc/apt/sources.list.d/pgadmin4.list
```

```bash
sudo apt update
```

```bash
sudo apt install pgadmin4-desktop -y
```

## Vérification du service PostgreSQL

```bash
sudo systemctl start postgresql
pg_lsclusters
```

---

# 4. Configuration PostgreSQL

## Ouvrir PostgreSQL

```bash
sudo -u postgres psql
```

## Définir le mot de passe PostgreSQL

```sql
ALTER USER postgres WITH PASSWORD 'mot_de_passe';
```

Résultat attendu :

```text
ALTER ROLE
```

## Création de la base JADE

```sql
CREATE DATABASE "JADE";
```

## Quitter PostgreSQL

```sql
\q
```

---

# 5. Configuration pgAdmin4

Dans pgAdmin4 :

- clic droit sur Servers ;
- sélectionner Register > Server.

## Onglet General

```text
PostgreSQL xx exemple : PostgreSQL 17 ou PostgreSQL 18
```

## Onglet Connection

```text
Host name/address : localhost
Username : postgres
Password : mot_de_passe
```

Puis cliquer sur Save.

---

# 6. Clonage du projet

```bash
mkdir TER
cd TER
```

```bash
git clone URL_DU_PROJET
```

```bash
cd abstrats
```

---

# 7. Création de l’environnement virtuel

## Création du venv

```bash
python3 -m venv .venv
```

## Activation du venv

```bash
source .venv/bin/activate
```

Lorsque l’environnement est activé, le terminal affiche :

```text
(.venv)
```

---

# 8. Installation des dépendances Python

## Installation des dépendances système

```bash
sudo apt install libpq-dev python3-dev build-essential -y
```

## Installation des packages Python

```bash
pip install -r requirements.txt
```

---

# 9. Configuration du fichier .env

## Copier le fichier modèle

```bash
cp .env_template .env
```

## Modifier le fichier .env

```bash
nano .env
```

## Contenu du fichier .env

```env
DB_NAME=JADE
DB_USER=postgres
DB_PASSWORD=mot_de_passe
DB_HOST=localhost
DB_PORT=5432
```

---

# 10. Vérification du dump PostgreSQL

Le fichier dump doit être présent dans :

```text
data/202605jade.dump
```

---

# 11. Restauration du dump PostgreSQL

Cette étape permet de restaurer automatiquement la base PostgreSQL à partir du fichier dump fourni dans le projet.

Le fichier dump contient déjà toute la base JADE avec les tables et les données nécessaires.

Si la base JADE est déjà restaurée dans PostgreSQL, il n’est pas nécessaire de refaire cette étape ; il suffit directement d’exécuter le pipeline.

La restauration du dump est uniquement nécessaire si la base JADE n’existe pas encore dans PostgreSQL.

## Vérification automatique de `pg_restore`

Le projet utilise :

```python
import shutil

PG_RESTORE_EXE = shutil.which("pg_restore")
```

Cela permet au projet de fonctionner automatiquement :

- sous Linux
- sous Windows
- sans chemin PostgreSQL fixe

## Exécution du restore

```bash
python -m src.admin.restore_dump
```

---

# 12. Lancement du pipeline

```bash
python -m src.implementation_abstrats.run_pipeline
```

---

