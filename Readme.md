# FML Challenge 2026 : la mobilité urbaine à travers les vélos en libre-service européens

Projet réalisé dans le cadre du cours **Fundamentals of Machine Learning (FML)**. Le sujet complet est dans [FML_Challenge_2026-1.pdf](FML_Challenge_2026-1.pdf).

## Objectif

Les données décrivent l'utilisation de systèmes de vélos en libre-service dans environ 185 villes européennes, d'août 2022 à juillet 2023.

Le but n'est pas seulement d'obtenir des modèles précis, mais de **découvrir des schémas de mobilité urbaine** et de montrer comment ils aident à **mieux comprendre et mieux exploiter** ces systèmes (par exemple, le rééquilibrage des vélos entre stations).

Le projet répond aux trois tâches du sujet :

| Tâche | Contenu | Notebooks |
|---|---|---|
| 3.1 Exploration et préparation | compréhension, nettoyage, fusion, statistiques, corrélations, visualisations | `01`, `02`, `03` |
| 3.2 Non supervisé | A : profils de trajets ; B : profils de stations (ACP + K-means) | `04`, `05` |
| 3.3 Supervisé | prédire le nombre de vélos disponibles dans une station **une heure plus tard** | `06` |

## Données

Les fichiers bruts ne sont **pas versionnés**. Copiez-les dans `data/raw/` :

| Fichier | Contenu | Clés |
|---|---|---|
| `cities.csv` | villes : centre, pays, fuseau horaire, règle de retour en station | `id` |
| `bike_types.csv` | types de vélos : propulsion (humaine / électrique), forme, autonomie | `id` |
| `bikes.csv` | vélos individuels | `id`, `bike_type_id` → `bike_types.id` |
| `stations.csv` | stations : position, nombre de bornes | `id`, `city_id` → `cities.id` |
| `stations_status.csv` | état des stations à un instant donné (vélos disponibles, bornes libres) | `station_id` → `stations.id`, `time` |
| `trips_small.csv` | trajets : début, durée, distance, coordonnées, batterie | `bike_id` → `bikes.id`, `city_id`, `station_id_start` / `station_id_end` |

Les relations entre les tables sont représentées dans [schema_relationel.png](schema_relationel.png).

Les heures sont en **UTC** dans les fichiers bruts. Elles sont converties en heure locale pendant la préparation.

## Structure du dépôt

```
FML-Challenge/
├── data/                     # non versionné
│   ├── raw/                  # CSV d'origine, jamais modifiés
│   ├── interim/              # tables nettoyées (parquet), une par fichier brut
│   └── processed/            # tables fusionnées, prêtes pour l'analyse (parquet)
├── models/
│   └── hgb_model.joblib      # modèle de prédiction entraîné (tâche 3.3)
├── notebooks/
│   ├── 01_exploration.ipynb          # compréhension de chaque table
│   ├── 02_preprocessing.ipynb        # nettoyage → interim/, fusion → processed/
│   ├── 03_eda.ipynb                  # statistiques, corrélations, visualisations
│   ├── 04_clustering_trips.ipynb     # 3.2 A : profils de trajets
│   ├── 05_clustering_stations.ipynb  # 3.2 B : profils de stations
│   └── 06_prediction.ipynb           # 3.3 : prédiction à +1 h
├── FML_Challenge_2026-1.pdf
├── schema_relationel.png
├── requirements.txt
└── Readme.md
```

## Installation

Testé avec Python 3.14.

```bash
git clone <url-du-depot>
cd FML-Challenge

python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

pip install -r requirements.txt
```

Pour utiliser l'environnement dans Jupyter / VS Code, sélectionnez le kernel du `.venv`, ou enregistrez-le :

```bash
python -m ipykernel install --user --name fml-challenge --display-name "Python (.venv - FML Challenge)"
```

## Exécution

Les notebooks s'exécutent **dans l'ordre**, chacun avec *Restart & Run All*. Chacun lit les fichiers produits par le précédent :

| Ordre | Notebook | Lit | Produit |
|---|---|---|---|
| 1 | `01_exploration` | `data/raw/` | — (analyse seulement) |
| 2 | `02_preprocessing` | `data/raw/` | `data/interim/*.parquet`, `data/processed/*.parquet` |
| 3 | `03_eda` | `data/processed/` | — |
| 4 | `04_clustering_trips` | `data/processed/trips.parquet` | — |
| 5 | `05_clustering_stations` | `data/processed/` | — |
| 6 | `06_prediction` | `data/processed/stations_status.parquet` | `models/hgb_model.joblib` |

`02_preprocessing` est le seul notebook obligatoire avant les autres. `01_exploration` peut être lancé à tout moment.

Le chargement des fichiers bruts (environ 1,2 million de trajets et 2,9 millions de relevés de stations) demande quelques Go de mémoire vive.

## Démarche

### 1. Exploration et préparation (`01` → `03`)
- Étude de chaque table : valeurs manquantes, doublons, plages de valeurs, liens entre tables.
- Nettoyage : suppression des colonnes vides, correction des valeurs impossibles, correction des fuseaux horaires à partir du pays, conversion en heure locale.
- Les cas douteux sont **marqués par un indicateur** (`is_virtual`, `battery_inconsistent`, `enough_snapshots`…) plutôt que supprimés.
- Fusion des tables, vérifiée par le nombre de lignes avant et après.
- Analyse exploratoire : rythmes d'utilisation, vélos électriques ou humains, batterie, corrélations, stations vides ou pleines.

### 2. Clustering (`04`, `05`)
Même démarche pour les trajets et pour les stations :

> choix des variables → log des variables asymétriques → standardisation → **ACP** (80 % de variance) → choix de k (coude + silhouette) → **K-means** → profil et interprétation des groupes

Pour les stations, les données sont d'abord agrégées en une ligne par station : activité, heures de départ, solde arrivées/départs, distance au centre, taux de vide.

### 3. Prédiction (`06`)
- Relevés transformés en **série horaire régulière** par station (remplissage limité à 6 h).
- Cible : nombre de vélos disponibles **dans 1 heure**. Variables : valeur actuelle, valeurs à −1 h et −24 h, heure, jour, week-end, capacité.
- Découpage **temporel** : entraînement jusqu'au 30 avril 2023, test de mai à juillet 2023.
- Comparaison avec une **référence naïve** (même valeur dans 1 h), une régression linéaire et un `HistGradientBoostingRegressor`. Métriques : MAE et RMSE.

### Évaluation finale
Le modèle sauvegardé est rechargé **sans réentraînement** :

```python
import joblib
model = joblib.load("models/hgb_model.joblib")
y_pred = model.predict(X_eval)   # X_eval construit avec les mêmes variables que dans 06_prediction
```

## Auteurs

Équipe FML Challenge 2026 — à compléter avec les noms des membres.
