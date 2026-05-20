# SQL employee management exercise — complete walkthrough

**Exercice**: Employee management system with PostgreSQL  
**Objectif**: Pratiquer CRUD operations (INSERT, SELECT, UPDATE, DELETE)  

---

## Contexte

Cet exercice permet de pratiquer les opérations SQL fondamentales en gérant une base de données d'employés. Vous apprendrez à:
- Créer une base de données et une table
- Insérer, consulter, modifier et supprimer des données
- Filtrer et trier les résultats
- Ajouter des colonnes à une table existante

---

# Setup Environnement

## Lancer PostgreSQL (Docker)

```bash
# Créer et lancer un container PostgreSQL
docker run --name some-postgres -e POSTGRES_PASSWORD=postgres -d postgres

# Se connecter à PostgreSQL
docker exec -it some-postgres psql -U postgres

# Résultat: postgres=# prompt
```

---

# Étape 1: Créer la base de données et la table

## 1.1 Créer la base de données

```sql
CREATE DATABASE company_db;
```

**Explication**:
- `CREATE DATABASE` crée une nouvelle base de données
- `company_db` est le nom de la base

**Résultat attendu**:
```
CREATE DATABASE
```

---

## 1.2 Se connecter à la base de données

```sql
\c company_db
```

**Explication**:
- `\c` est un meta-command PostgreSQL (commence par backslash)
- Change la connexion vers la base `company_db`

**Résultat attendu**:
```
You are now connected to database "company_db" as user "postgres".
company_db=#
```

**Important**: Le prompt doit maintenant afficher `company_db=#` au lieu de `postgres=#`

---

## 1.3 Créer la table employees

```sql
CREATE TABLE employees (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    position TEXT,
    salary INT
);
```

**Explication ligne par ligne**:

| Ligne | Signification |
|-------|---------------|
| `CREATE TABLE employees` | Crée une nouvelle table nommée `employees` |
| `id SERIAL PRIMARY KEY` | Colonne `id`: entier auto-incrémenté, identifiant unique |
| `name TEXT NOT NULL` | Colonne `name`: texte obligatoire (ne peut pas être NULL) |
| `position TEXT` | Colonne `position`: texte optionnel |
| `salary INT` | Colonne `salary`: entier optionnel |

**Résultat attendu**:
```
CREATE TABLE
```

---

## Vérifier la création de la table

```sql
\dt
```

**Résultat attendu**: Liste les tables dans la base
```
         List of relations
 Schema |   Name    | Type  |  Owner
--------+-----------+-------+----------
 public | employees | table | postgres
(1 row)
```

---

# Étape 2: Remplir la table (INSERT)

## 2.1 Insérer trois employés

```sql
INSERT INTO employees (name, position, salary) VALUES 
('Alice Smith', 'Software Engineer', 75000),
('Bob Johnson', 'Data Analyst', 68000),
('Charlie Lee', 'HR Manager', 72000);
```

**Explication**:
- `INSERT INTO employees` insère des données dans la table
- `(name, position, salary)` spécifie les colonnes à remplir
- `VALUES (...)` contient les données
- Les trois employés sont insérés en une seule requête
- Les IDs (1, 2, 3) sont auto-générés par `SERIAL`

**Résultat attendu**:
```
INSERT 0 3
```
(0 = oid inutile en moderne PostgreSQL, 3 = nombre de lignes insérées)

---

## 2.2 Vérifier les données insérées

```sql
SELECT * FROM employees;
```

**Résultat attendu**:
```
 id |    name     |      position      | salary
----+-------------+--------------------+--------
  1 | Alice Smith | Software Engineer  |  75000
  2 | Bob Johnson | Data Analyst       |  68000
  3 | Charlie Lee | HR Manager         |  72000
(3 rows)
```

### Explications du résultat

| Colonne | Valeur | Type |
|---------|--------|------|
| `id` | 1, 2, 3 | SERIAL (auto-incrémenté) |
| `name` | Prénoms et noms | TEXT |
| `position` | Titres de poste | TEXT |
| `salary` | Salaires | INT |

---

# Étape 3: Modifier les données (UPDATE & DELETE)

## 3.1 Augmenter le salaire d'Alice

```sql
UPDATE employees SET salary = 80000 WHERE name = 'Alice Smith';
```

**Explication**:
- `UPDATE employees` modifie la table
- `SET salary = 80000` change le salaire à 80000
- `WHERE name = 'Alice Smith'` spécifie QUI est modifié
- **Important**: Sans `WHERE`, TOUS les salaires seraient changés!

**Résultat attendu**:
```
UPDATE 1
```
(1 = nombre de lignes modifiées)

---

## 3.2 Vérifier le changement

```sql
SELECT * FROM employees WHERE name = 'Alice Smith';
```

**Résultat attendu**:
```
 id |    name     |     position      | salary
----+-------------+-------------------+--------
  1 | Alice Smith | Software Engineer |  80000
(1 row)
```

---

## 3.3 Supprimer Bob (démission)

```sql
DELETE FROM employees WHERE name = 'Bob Johnson';
```

**Explication**:
- `DELETE FROM employees` supprime des lignes
- `WHERE name = 'Bob Johnson'` spécifie QUI est supprimé
- **DANGER**: Sans `WHERE`, TOUTES les lignes seraient supprimées!

**Résultat attendu**:
```
DELETE 1
```
(1 = nombre de lignes supprimées)

---

## 3.4 Vérifier la suppression

```sql
SELECT * FROM employees;
```

**Résultat attendu**:
```
 id |    name     |   position   | salary
----+-------------+--------------+--------
  1 | Alice Smith | Software Engineer |  80000
  3 | Charlie Lee | HR Manager   |  72000
(2 rows)
```

**Important**: Bob n'est plus dans la table. Note que Charlie a toujours l'ID 3 (les IDs ne se renumérote pas).

---

# Étape 4: Filtrer et Trier (WHERE & ORDER BY)

## 4.1 Afficher les employés gagnant plus de 70 000

```sql
SELECT * FROM employees WHERE salary > 70000;
```

**Explication**:
- `WHERE salary > 70000` filtre les résultats
- Retourne seulement les lignes où le salaire dépasse 70000

**Résultat attendu**:
```
 id |    name     |      position      | salary
----+-------------+--------------------+--------
  1 | Alice Smith | Software Engineer  |  80000
  3 | Charlie Lee | HR Manager         |  72000
(2 rows)
```

---

## 4.2 Classer par salaire décroissant

```sql
SELECT * FROM employees ORDER BY salary DESC;
```

**Explication**:
- `ORDER BY salary` trie par colonne salary
- `DESC` = descending (décroissant, du plus haut au plus bas)
- `ASC` = ascending (croissant, du plus bas au plus haut)

**Résultat attendu**:
```
 id |    name     |      position      | salary
----+-------------+--------------------+--------
  1 | Alice Smith | Software Engineer  |  80000
  3 | Charlie Lee | HR Manager         |  72000
(2 rows)
```

**Ordre**: Alice (80000) avant Charlie (72000)

---

# Étape 5: Ajouter une colonne (ALTER TABLE)

## 5.1 Ajouter la colonne is_remote

```sql
ALTER TABLE employees ADD COLUMN is_remote BOOLEAN DEFAULT FALSE;
```

**Explication**:
- `ALTER TABLE employees` modifie la structure de la table
- `ADD COLUMN is_remote` ajoute une nouvelle colonne
- `BOOLEAN` type booléen (TRUE/FALSE)
- `DEFAULT FALSE` définit la valeur par défaut à FALSE pour toutes les lignes

**Résultat attendu**:
```
ALTER TABLE
```

---

## 5.2 Vérifier la colonne ajoutée

```sql
SELECT * FROM employees;
```

**Résultat attendu**:
```
 id |    name     |      position      | salary | is_remote
----+-------------+--------------------+--------+-----------
  1 | Alice Smith | Software Engineer  |  80000 | f
  3 | Charlie Lee | HR Manager         |  72000 | f
(2 rows)
```

**Note**: `f` = FALSE, `t` = TRUE (PostgreSQL utilise f/t pour afficher)

---

## 5.3 Optionnel: Modifier la valeur is_remote

```sql
UPDATE employees SET is_remote = TRUE WHERE name = 'Alice Smith';
SELECT * FROM employees;
```

**Résultat attendu**:
```
 id |    name     |      position      | salary | is_remote
----+-------------+--------------------+--------+-----------
  1 | Alice Smith | Software Engineer  |  80000 | t
  3 | Charlie Lee | HR Manager         |  72000 | f
(2 rows)
```

---

# Résumé des opérations CRUD

## CREATE (Créer)

```sql
CREATE TABLE employees (...);        -- Créer une table
INSERT INTO employees (...) VALUES;  -- Insérer des données
ALTER TABLE employees ADD COLUMN;    -- Ajouter une colonne
```

## READ (Lire)

```sql
SELECT * FROM employees;                    -- Tous les employés
SELECT * FROM employees WHERE salary > 70000;  -- Filtrés
SELECT * FROM employees ORDER BY salary DESC;  -- Triés
```

## UPDATE (Modifier)

```sql
UPDATE employees SET salary = 80000 WHERE name = 'Alice Smith';
```

## DELETE (Supprimer)

```sql
DELETE FROM employees WHERE name = 'Bob Johnson';
```

---

# Concepts Clés Appris

## 1. Types de données

| Type | Exemple | Usage |
|------|---------|-------|
| `SERIAL` | 1, 2, 3 | Auto-increment ID |
| `TEXT` | "Alice Smith" | Texte variable |
| `INT` | 75000 | Entiers |
| `BOOLEAN` | TRUE, FALSE | Booléen (f/t en affichage) |

## 2. Constraints

| Constraint | Signification |
|------------|---------------|
| `PRIMARY KEY` | Identifiant unique, non-NULL |
| `NOT NULL` | Valeur obligatoire |
| `DEFAULT` | Valeur par défaut |

## 3. Meta-commands (\...)

| Commande | Fonction |
|----------|----------|
| `\l` | Lister les bases de données |
| `\c dbname` | Se connecter à une base |
| `\dt` | Lister les tables |
| `\d tablename` | Décrire une table |
| `\q` | Quitter |

## 4. Bonnes pratiques

### ✓ À faire

```sql
-- Vérifier avant de supprimer/modifier
SELECT * FROM employees WHERE name = 'Bob Johnson';  -- Vérifier
DELETE FROM employees WHERE name = 'Bob Johnson';     -- Puis supprimer

-- Utiliser WHERE systématiquement
UPDATE employees SET salary = 80000 WHERE id = 1;

-- Utiliser des colonnes spécifiques
SELECT name, salary FROM employees;  -- Pas SELECT *
```

### ✗ À éviter

```sql
-- JAMAIS de UPDATE/DELETE sans WHERE!
UPDATE employees SET salary = 80000;  -- Tous les salaires changent!
DELETE FROM employees;                 -- Tout est supprimé!

-- JAMAIS de SELECT * sur grandes tables
SELECT * FROM employees;  -- OK ici, mais mauvais sur millions de lignes
```

---

# État final de la table

Après l'exercice complet:

```
 id |    name     |      position      | salary | is_remote
----+-------------+--------------------+--------+-----------
  1 | Alice Smith | Software Engineer  |  80000 | f
  3 | Charlie Lee | HR Manager         |  72000 | f
(2 rows)
```

### Résumé des changements

| Étape | Action | Détail |
|-------|--------|--------|
| 1 | Créer table | 3 colonnes: id, name, position, salary |
| 2 | Insérer 3 employés | Alice, Bob, Charlie |
| 3a | Augmenter Alice | Salaire 75000 → 80000 |
| 3b | Supprimer Bob | Démission |
| 4a | Filtrer | Afficher salary > 70000 |
| 4b | Trier | ORDER BY salary DESC |
| 5 | Ajouter colonne | is_remote BOOLEAN DEFAULT FALSE |

---


Prêt pour les prochains exercices SQL avancés! 🚀
