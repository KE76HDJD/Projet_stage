# Modélisation des Courbes IDF — Station de Kara (Nord Togo)

[![Python](https://img.shields.io/badge/Python-3.9+-blue.svg)](https://www.python.org/)
[![Jupyter Notebook](https://img.shields.io/badge/Jupyter-Notebook-orange.svg)](https://jupyter.org/)
[![Status](https://img.shields.io/badge/Statut-Terminé%20%2F%20Validé-success.svg)]()

Ce dépôt contient l'ensemble des scripts, données et résultats du projet de modélisation des **courbes Intensité-Durée-Fréquence (IDF)** pour la station météorologique de Kara, au Togo. Ces courbes constituent un outil fondamental pour le dimensionnement hydrologique et la gestion des risques d'inondation dans la région.


## 📌 Objectif du Projet

L'objectif principal est de formaliser la relation entre l'intensité de la pluie ($I$), sa durée ($D$) et sa période de retour ($T$) via le modèle mathématique de Montana :

$$I(D,T) = a(T) \times D^{b(T)}$$

Pour pallier le manque de données pluviographiques historiques à long terme (seulement disponibles de 2004 à 2014), ce projet déploie une approche hybride basée sur le **Machine Learning** afin de reconstituer les intensités maximales de 1980 à 2003 à partir des données pluviométriques journalières historiques.

---

## 📂 Architecture du Projet

Le projet est structuré comme suit afin de garantir la reproductibilité complète de la pipeline de traitement :

```text
Projet_stage/
│
├── Donnees/                          # Données brutes sources
│   ├── KARA PLUIE JOURNALIERE.xls    # Pluviomètre journalier (1980-2014)
│   └── INTENSITES DE PLUIE -KARA.xlsx # Pluviographe intensités (2004-2014)
│
├── data/                             # Données intermédiaires et finales (CSV)
│   ├── pluviometre_clean.csv         # Données journalières nettoyées
│   ├── pluviographe_clean.csv        # Intensités nettoyées
│   ├── tableau_entrainement.csv      # Dataset de fusion (période commune)
│   ├── serie_complete.csv            # Série temporelle finale reconstituée
│   ├── tableau_idf.csv               # Matrice finale des intensités IDF
│   └── parametres_idf.csv            # Paramètres de Montana calculés
│
├── figures/                          # Visualisations graphiques exportées
│   ├── figures_n1/ à n8/             # Graphiques triés par étape/notebook
│
├── n1_exploration_pluviometre.ipynb  # Analyse descriptive du pluviomètre
├── n2_exploration_pluviographe.ipynb # Analyse descriptive du pluviographe
├── n3_extraction_nettoyage.ipynb     # Parsing et imputation des données
├── n4_fusion_preparation.ipynb       # Alignement temporel et Feature Engineering
├── n5_modelisation.ipynb             # Entraînement ML & Reconstruction historique
├── n6_extremes.ipynb                 # Ajustements aux lois statistiques (GEV/Gumbel)
├── n7_courbes_idf.ipynb              # Extraction des paramètres de Montana & IDF
├── n8_validation_modele.ipynb        # Validation croisée et analyse des résidus
│
├── README.md                         # Ce fichier (Documentation principale)
├── README_PIPELINE.md                # Documentation technique détaillée
└── PLAN_FICHIER.md                   # Plan de structure initial
