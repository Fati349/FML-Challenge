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

