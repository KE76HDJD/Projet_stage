# Plan du Projet — Modélisation des Courbes IDF
## Station de Kara, Nord Togo

**Auteur :** AY2K  
**Encadreur :** Prof. Titembaye Donald  
**Établissement :** École Polytechnique de Lomé (EPL)  
**Date :** 09 Juin 2026

---

## 1. Objectif du Projet

Ce projet vise à **modéliser les courbes IDF** (Intensité-Durée-Fréquence) pour la  
station météorologique de Kara, au Togo. Les courbes IDF sont utilisées pour le  
dimensionnement d'ouvrages hydrauliques (réseaux d'assainissement, bassins de  
rétention, ponts, etc.).

---

## 2. Structure du Projet

```
Projet_stage/
│
├── Donnees/                          # Données brutes (non modifiées)
│   ├── KARA PLUIE JOURNALIERE.xls    # Pluviomètre journalier (1980-2014)
│   └── INTENSITES DE PLUIE -KARA.xlsx # Pluviographe intensités (2004-2014)
│
├── data/                             # Données traitées (CSV)
│   ├── pluviometre_clean.csv         # Nettoyé par n3
│   ├── pluviographe_clean.csv        # Nettoyé par n3
│   ├── tableau_entrainement.csv      # Fusionné par n4
│   ├── serie_complete.csv            # Reconstitué par n5
│   ├── tableau_idf.csv               # Tableau IDF par n7
│   └── parametres_idf.csv            # Paramètres Montana par n7
│
├── figures/                          # Figures par notebook
│   ├── figures_n1/                   # 5 figures exploration pluviomètre
│   ├── n2/                           # 3 figures exploration pluviographe
│   ├── n3/                           # 3 figures nettoyage
│   ├── n4/                           # 3 figures fusion
│   ├── n5/                           # 4 figures modélisation
│   ├── n6/                           # 4 figures extrêmes
│   ├── n7/                           # 3 figures courbes IDF
│   └── n8/                           # 4 figures validation
│
├── n1_exploration_pluviometre.ipynb  # Exploration pluviomètre
├── n2_exploration_pluviographe.ipynb # Exploration pluviographe
├── n3_extraction_nettoyage.ipynb     # Extraction et nettoyage
├── n4_fusion_preparation.ipynb       # Fusion des données
├── n5_modelisation.ipynb             # Modélisation (régression)
├── n6_extremes.ipynb                 # Lois statistiques extrêmes
├── n7_courbes_idf.ipynb              # Courbes IDF finales
├── n8_validation_modele.ipynb        # Validation du modèle
│
├── README_PIPELINE.md                # Documentation pipeline
└── PLAN_FICHIER.md                   # Ce fichier
```

---

## 3. Description des Notebooks

### Notebook 1 — Exploration du Pluviomètre
**Fichier :** `n1_exploration_pluviometre.ipynb`  
**Entrée :** `Donnees/KARA PLUIE JOURNALIERE.xls`  
**Statut :** ✅ Confirmé bon par l'utilisateur

| Contenu | Description |
|---------|-------------|
| Totaux annuels | Pluie totale par année (1980-2014) |
| Saisonnalité | Distribution mensuelle moyenne |
| Distribution | Histogramme des pluies journalières |
| Jours de pluie | Nombre de jours pluvieux par an |
| Max annuel | Pluie maximale journalière par année |

**5 figures** dans `figures/figures_n1/`

---

### Notebook 2 — Exploration du Pluviographe
**Fichier :** `n2_exploration_pluviographe.ipynb`  
**Entrée :** `Donnees/INTENSITES DE PLUIE -KARA.xlsx`  
**Statut :** ✅ Confirmé bon par l'utilisateur

| Contenu | Description |
|---------|-------------|
| Intensités-durées | Relation I-D pour chaque événement |
| Événements extrêmes | Top 10 des pluies les plus intenses |
| Distribution | Histogramme des intensités par durée |

**3 figures** dans `figures/n2/`

---

### Notebook 3 — Extraction et Nettoyage
**Fichier :** `n3_extraction_nettoyage.ipynb`  
**Entrée :** Fichiers bruts `Donnees/`  
**Sortie :** `data/pluviometre_clean.csv`, `data/pluviographe_clean.csv`  
**Statut :** ✅ Exécuté et validé

| Contenu | Description |
|---------|-------------|
| Extraction pluviomètre | Parsing du fichier .xls, reconstruction des dates |
| Extraction pluviographe | Parsing BLOCS_MOIS, colonnes hm/dm/hs/ds |
| Nettoyage | Détection outliers (IQR), imputation médiane mensuelle |
| Validation | Vérification après traitement |

**3 figures** dans `figures/n3/`

**Détails d'extraction :**
- Pluviomètre : 8182 lignes, 1980-01-06 → 2014-12-31
- Pluviographe : 1115 lignes, 2004 → 2014 (86 durées uniques)

---

### Notebook 4 — Fusion et Préparation
**Fichier :** `n4_fusion_preparation.ipynb`  
**Entrée :** `data/pluviometre_clean.csv`, `data/pluviographe_clean.csv`  
**Sortie :** `data/tableau_entrainement.csv`  
**Statut :** ✅ Exécuté et validé

| Contenu | Description |
|---------|-------------|
| Jointure | Inner join date pour la période commune (2004-2014) |
| Features | mois (1-12), saison (sèche/pluies) |
| Statistiques | Résumé des données fusionnées |

**3 figures** dans `figures/n4/`

**Résultat :** 617 lignes, 2004-06-03 → 2014-10-31

---

### Notebook 5 — Modélisation
**Fichier :** `n5_modelisation.ipynb`  
**Entrée :** `data/tableau_entrainement.csv`, `data/pluviometre_clean.csv`  
**Sortie :** `data/serie_complete.csv`  
**Statut :** ✅ Exécuté et validé

| Contenu | Description |
|---------|-------------|
| Features | pluie_mm, log_pluie, mois_sin, mois_cos |
| Target | log(1 + intensite_max_mm_h) |
| Modèles | LinearRegression, RandomForest, GradientBoosting |
| Validation | K-Fold 5-fold, R² et RMSE |
| Reconstruction | Prédiction 1980-2003 via meilleur modèle |

**4 figures** dans `figures/n5/`

**Résultat :** 3012 lignes, 1980-2014  
**Répartition :** 82.9% reconstitué, 17.1% observé

---

### Notebook 6 — Extremes et Lois Statistiques
**Fichier :** `n6_extremes.ipynb`  
**Entrée :** `data/serie_complete.csv`  
**Statut :** ✅ Exécuté (bootstrap n=200)

| Contenu | Description |
|---------|-------------|
| Maxima annuels | Max d'intensité par année |
| Ajustement | GEV, Gumbel, Lognormale |
| Sélection | Critère AIC minimal |
| Valeurs de retour | T = 2, 5, 10, 20, 50, 100 ans |
| Bootstrap | IC 95% (n=200) |

**4 figures** dans `figures/n6/`

---

### Notebook 7 — Courbes IDF
**Fichier :** `n7_courbes_idf.ipynb`  
**Entrée :** `data/pluviographe_clean.csv`, `data/serie_complete.csv`  
**Sortie :** `data/tableau_idf.csv`, `data/parametres_idf.csv`  
**Statut :** ✅ Exécuté et validé

| Contenu | Description |
|---------|-------------|
| Maxima par durée | Max annuel pour chaque durée |
| Gumbel par durée | Ajustement par durée, calcul des T |
| Formule Montana | I(D,T) = a(T) × D^b(T) |
| Paramètres a, b | Par période de retour |
| Enveloppe IC | Bootstrap courbes IDF |

**3 figures** dans `figures/n7/`

**Résultat :**

| T (ans) | a | b | R² |
|---------|------|--------|-------|
| 2 | 177.2 | -0.599 | 0.777 |
| 5 | 338.9 | -0.599 | 0.777 |
| 10 | 450.2 | -0.599 | 0.777 |
| 20 | 561.5 | -0.599 | 0.777 |
| 50 | 715.2 | -0.599 | 0.777 |
| 100 | 828.4 | -0.599 | 0.777 |

---

### Notebook 8 — Validation du Modèle
**Fichier :** `n8_validation_modele.ipynb`  
**Entrée :** `data/tableau_entrainement.csv`  
**Statut :** ✅ Exécuté et validé

| Contenu | Description |
|---------|-------------|
| Holdout 80/20 | 1 split aléatoire |
| Leave-One-Out | Chaque point testé (LR) / K-20 (RF, GB) |
| K-Fold (k=5) | 5 plis moyennés |
| Métriques | R², RMSE, MAE, MAPE |
| Résidus | Histogramme, scatter, Q-Q plot |

**4 figures** dans `figures/n8/`

---

## 4. Données CSV

| Fichier | Lignes | Colonnes | Période | Description |
|---------|--------|----------|---------|-------------|
| `pluviometre_clean.csv` | 8182 | date, pluie_mm, flag_outlier, flag_manquant | 1980-2014 | Pluies journalières nettoyées |
| `pluviographe_clean.csv` | 1115 | date, duree_min, intensite_mm_h, flag_outlier | 2004-2014 | Intensités par durée nettoyées |
| `tableau_entrainement.csv` | 617 | date, pluie_mm, intensite_max_mm_h, mois, saison | 2004-2014 | Données d'entraînement |
| `serie_complete.csv` | 3012 | date, intensite_max_mm_h, source, pluie_mm | 1980-2014 | Série complète reconstituée |
| `tableau_idf.csv` | 50 | duree_min, T2, T5, T10, T20, T50, T100 | - | Matrice intensités IDF |
| `parametres_idf.csv` | 6 | T, a, b, R² | - | Paramètres Montana |

---

## 5. Pipeline d'Exécution

```
n1 (exploration) ────────────────────────────────┐
n2 (exploration) ────────────────────────────────┤
                                                  │
Données brutes ──→ n3 (nettoyage) ──→ pluviometre_clean.csv
                                        pluviographe_clean.csv
                                                  │
                                                  ▼
                                        n4 (fusion) ──→ tableau_entrainement.csv
                                                          │
                                                          ▼
                                        n5 (modélisation) ──→ serie_complete.csv
                                                              │
                                                              ▼
                                        n6 (extrêmes) ──→ lois, valeurs de retour
                                                              │
                                                              ▼
n7 (courbes IDF) ──→ tableau_idf.csv, parametres_idf.csv
                          
n8 (validation) ──→ évaluation du modèle
```

**Commande pour tout exécuter :**
```bash
for nb in n3 n4 n5 n6 n7 n8; do
  ~/anaconda3/bin/jupyter nbconvert --to notebook --execute ${nb}_*.ipynb --output ${nb}_*.ipynb --ExecutePreprocessor.timeout=300
done
```

---

## 6. Fichiers PDF

| Fichier | Description |
|---------|-------------|
| `Explication_complet_claire_du_projet.pdf` | Explication détaillée du projet |
| `Cahier_de_charges_IDF_Kara.pdf` | Cahier des charges officiel |
| `cahier_texte.txt` | Texte extrait du cahier |
| `explication_texte.txt` | Texte extrait de l'explication |

---

*Document généré le 09/06/2026*
