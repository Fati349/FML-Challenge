# FML Challenge : la mobilité urbaine à travers les vélos en libre-service européens

Projet réalisé dans le cadre du cours **Fundamentals of Machine Learning (FML)**.

## Objectif

Le jeu de données décrit l'utilisation quotidienne de systèmes de vélos en libre-service dans plusieurs villes européennes.

Le but **n'est pas seulement** d'obtenir des modèles de machine learning précis. Il s'agit surtout de :

1. **découvrir des schémas pertinents de mobilité urbaine** (quand, où et comment les vélos sont utilisés) ;
2. **montrer en quoi ces schémas aident à mieux comprendre et à mieux exploiter** les systèmes de vélos partagés en Europe (rééquilibrage des stations, dimensionnement, planification).

Un modèle ne compte que s'il apporte un éclairage interprétable ou une utilité opérationnelle concrète.

## Données

| Fichier | Contenu | Clés probables |
|---|---|---|
| `cities.csv` | Villes équipées d'un système de vélos partagés, avec leur position géographique et leur pays | `city_id` |
| `bikes.csv` | Vélos individuels, chacun rattaché à un type | `bike_id` → `bike_type_id` |
| `bike_types.csv` | Caractéristiques des vélos (propulsion humaine ou électrique, etc.) | `bike_type_id` |
| `stations.csv` | Stations : localisation et capacité | `station_id` → `city_id` |
| `station_status.csv` | Disponibilité des vélos dans une station à un instant donné | `station_id` + horodatage |
| `trips.csv` | Trajets : heures et coordonnées géographiques de départ et d'arrivée | `bike_id`, coordonnées, horodatages |

> Les noms de colonnes ci-dessus sont des hypothèses : à vérifier lors de la première exploration et à corriger ici.

### Schéma relationnel (simplifié)

```
cities ──< stations ──< station_status
                │
bike_types ──< bikes ──< trips  (départ / arrivée géolocalisés)
```

Les données brutes ne sont **pas versionnées** dans Git. Placez-les dans `data/raw/` (voir ci-dessous).

## Structure du dépôt

```
FML-Challenge/
├── data/
│   ├── raw/            # CSV d'origine (non versionnés)
│   └── processed/      # Données nettoyées / features (non versionnées)
├── notebooks/          # Exploration et analyses (préfixées 01_, 02_, …)
├── src/                # Code réutilisable : chargement, nettoyage, features, modèles
├── reports/
│   └── figures/        # Graphiques produits pour le rapport
├── requirements.txt
└── README.md
```

## Démarrage

```bash
# 1. Cloner le dépôt
git clone <url-du-depot>
cd FML-Challenge

# 2. Créer un environnement virtuel
python -m venv .venv
# Windows
.venv\Scripts\activate
# macOS / Linux
source .venv/bin/activate

# 3. Installer les dépendances
pip install -r requirements.txt

# 4. Copier les CSV du challenge dans data/raw/
```

Bibliothèques suggérées : `pandas`, `numpy`, `scikit-learn`, `matplotlib`, `seaborn`, `geopandas` / `folium` (cartographie), `jupyter`.

## Démarche proposée

### 1. Exploration et qualité des données
- Volume, période couverte et nombre de villes, de stations et de vélos.
- Valeurs manquantes, doublons, horodatages incohérents (arrivée avant départ, durées aberrantes).
- Trajets aux coordonnées impossibles ou à distance nulle.

### 2. Feature engineering
- Durée et distance des trajets (haversine), vitesse moyenne.
- Variables temporelles : heure, jour de la semaine, week-end ou jours fériés, saison.
- Rattachement des points de départ et d'arrivée à la station la plus proche.
- Taux de remplissage des stations (`vélos disponibles / capacité`).
- Type de vélo (mécanique ou électrique) joint aux trajets.

### 3. Pistes d'analyse
- **Rythmes temporels** : pics domicile-travail vs usage de loisir, différences entre semaine et week-end.
- **Comparaison entre villes et pays** : intensité d'usage, durée des trajets, part de vélos électriques.
- **Typologie des stations** (clustering) : stations résidentielles « émettrices » le matin, stations de bureaux « réceptrices », stations touristiques.
- **Flux origine-destination** : corridors principaux, asymétries de flux qui expliquent les besoins de rééquilibrage.
- **Électrique vs mécanique** : les vélos électriques servent-ils à des trajets plus longs ou plus vallonnés ?
- **Prédiction de disponibilité** : anticiper les stations vides ou pleines pour guider le rééquilibrage.
- **Détection d'anomalies** : stations défaillantes, vélos inutilisés, trajets atypiques.

### 4. Modélisation
- Partir d'une baseline simple (moyenne historique, régression linéaire) avant les modèles plus complexes.
- Utiliser une validation **temporelle** (pas de mélange aléatoire passé/futur) et, si pertinent, une validation entre villes.
- Privilégier l'interprétabilité : importance des variables, SHAP, visualisation des clusters.

### 5. Restitution
Chaque résultat doit répondre à la question : **qu'apprend-on sur la mobilité, et que pourrait en faire un opérateur ?**

## Conventions de travail

- Une branche par analyse ou fonctionnalité, fusionnée dans `main` via pull request.
- Notebooks numérotés (`01_exploration.ipynb`, `02_features.ipynb`, …), sorties vidées avant commit si elles sont volumineuses.
- Le code réutilisé dans plusieurs notebooks passe dans `src/`.
- Aucune donnée brute ni aucun fichier lourd dans Git (voir `.gitignore`).

