# Pipeline d'Analyse Pluviométrique — Station de Kara, Togo
## Documentation complète du projet

---

## 🎯 Objectif général

Produire les **courbes IDF (Intensité-Durée-Fréquence)** pour le dimensionnement d'ouvrages hydrauliques à Kara (Togo).

**Problème central :** le pluviographe (qui mesure les intensités à pas fins) ne couvre que 2004–présent, alors que le pluviomètre (pluies journalières) couvre 1980–présent. Il faut donc **reconstituer** les intensités manquantes de 1980 à 2003 par modélisation statistique, puis établir les courbes IDF sur la période complète.

---

## 📂 Structure du projet

```
Projet_stage/
├── Donnees/
│   ├── KARA PLUIE JOURNALIERE.xls        ← pluviomètre (pluies journalières, ~1980–présent)
│   └── INTENSITES DE PLUIE -KARA.xlsx    ← pluviographe (intensités à pas fins, 2004–présent)
│
├── data/                                 ← CSV produits par les notebooks
│   ├── pluviometre_clean.csv             ← produit par n3
│   ├── pluviographe_clean.csv            ← produit par n3
│   ├── tableau_entrainement.csv          ← produit par n4
│   ├── serie_complete.csv                ← produit par n5
│   ├── tableau_idf.csv                   ← produit par n7
│   └── parametres_idf.csv               ← produit par n7 (livrable final)
│
├── figures/
│   ├── n1/   n2/   n3/   n4/   n5/   n6/   n7/   ← figures par notebook
│
├── n1_exploration_pluviometre.ipynb
├── n2_exploration_pluviographe.ipynb
├── n3_extraction_nettoyage.ipynb
├── n4_fusion_preparation.ipynb
├── n5_modelisation.ipynb
├── n6_extremes.ipynb
└── n7_courbes_idf.ipynb
```

---

## 🔁 Flux de données (pipeline)

```
Donnees/KARA PLUIE JOURNALIERE.xls
        │
        ├──► n1 (exploration)
        │
        └──► n3 ──► pluviometre_clean.csv ──► n4 ──► tableau_entrainement.csv
                                                │
Donnees/INTENSITES DE PLUIE -KARA.xlsx         │
        │                                       │
        ├──► n2 (exploration)                   │
        │                                       ▼
        └──► n3 ──► pluviographe_clean.csv ──► n4
                                                │
                                                ▼
                                          n5 (modélisation)
                                                │
                                                ▼
                                        serie_complete.csv
                                         (1980–présent)
                                                │
                               ┌────────────────┴────────────────┐
                               ▼                                  ▼
                          n6 (extrêmes)                     n7 (IDF)
                        valeurs de retour              courbes IDF + Montana
```

---

## 📒 Description détaillée de chaque notebook

### n1 — Exploration brute du pluviomètre
**Entrée :** `Donnees/KARA PLUIE JOURNALIERE.xls`

**Ce que fait ce notebook :**
- Charge le fichier brut (format bloc : année × mois × jours 1..31)
- Reconstruit la série temporelle journalière
- Calcule les statistiques descriptives (moyenne, médiane, max, etc.)
- Calcule les statistiques annuelles (cumul, jours de pluie, maximum annuel)
- Identifie les lacunes temporelles

**Figures produites dans `figures/n1/` :**
| Fichier | Description |
|---|---|
| `fig1_totaux_annuels.png` | Barplot des cumuls annuels avec ligne de moyenne |
| `fig2_saisonnalite.png` | Boxplot mensuel — cycle saisonnier |
| `fig3_distribution.png` | Histogramme + KDE des pluies journalières |
| `fig4_jours_pluie.png` | Nombre de jours pluvieux par an |
| `fig5_max_annuel.png` | Série temporelle des maxima annuels journaliers |

---

### n2 — Exploration brute du pluviographe
**Entrée :** `Donnees/INTENSITES DE PLUIE -KARA.xlsx`

**Ce que fait ce notebook :**
- Charge le fichier multi-feuilles (1 feuille = 1 année, 2004–présent)
- Extrait les événements pluvieux : pour chaque date, les intensités à 14 durées (5, 10, 15, 20, 30, 45, 60, 90, 120, 180, 240, 360, 720, 1440 min)
- Calcule les statistiques par durée
- Identifie les top 10 événements extrêmes

**Figures produites dans `figures/n2/` :**
| Fichier | Description |
|---|---|
| `fig1_structure_donnees.png` | Heatmap de disponibilité temporelle (année × mois) |
| `fig2_intensites_durees.png` | Courbe moyenne/médiane/max par durée (axe log) |
| `fig3_evenements_extremes.png` | Barplot horizontal des 10 événements les plus intenses |
| `fig4_distribution_intensites.png` | Histogramme + loi log-normale + Q-Q plot |

---

### n3 — Extraction et nettoyage
**Entrée :** fichiers bruts (`Donnees/`)

**Ce que fait ce notebook :**
Applique le protocole de nettoyage complet sur les deux sources :

| Étape | Règle appliquée |
|---|---|
| Valeurs négatives | Converties en NaN |
| Doublons | Supprimés (log du nombre) |
| Valeurs aberrantes | Détection IQR × 3 → flag `flag_outlier=True` (conservées) |
| Valeurs manquantes | Si taux < 15% : imputation par **médiane mensuelle** ; sinon conservées avec flag |

**Figures produites dans `figures/n3/` :**
| Fichier | Description |
|---|---|
| `fig1_avant_apres_pluviometre.png` | Histogramme avant/après nettoyage (pluviomètre) |
| `fig2_avant_apres_pluviographe.png` | Histogramme avant/après nettoyage (pluviographe, durée 60 min) |
| `fig3_completude.png` | Taux de complétude par année et par source |

**Livrables CSV :**
- `data/pluviometre_clean.csv` — colonnes : `[date, pluie_mm, flag_outlier, flag_manquant]`
- `data/pluviographe_clean.csv` — colonnes : `[date, duree_min, intensite_mm_h, flag_outlier]`

---

### n4 — Fusion et préparation du tableau d'entraînement
**Entrée :** `data/pluviometre_clean.csv` + `data/pluviographe_clean.csv`

**Ce que fait ce notebook :**
- Calcule l'intensité max journalière (toutes durées confondues) depuis le pluviographe
- Identifie la période commune aux deux sources
- Effectue une jointure interne (`inner join`) sur la date
- Ajoute les variables temporelles : `mois` et `saison` (sèche / transition / pluies)
- Calcule la corrélation entre pluie journalière et intensité max (Pearson, version log-log)

**Fallback :** si la période commune a moins de 10 jours, utilise toute la période disponible.

**Figures produites dans `figures/n4/` :**
| Fichier | Description |
|---|---|
| `fig1_periodes_communes.png` | Diagramme de Gantt des périodes de couverture |
| `fig2_correlation.png` | Nuage de points pluie vs intensité (linéaire + log-log) |
| `fig3_tableau_entrainement.png` | Distributions des variables + répartition saisonnière |

**Livrable CSV :**
- `data/tableau_entrainement.csv` — colonnes : `[date, pluie_mm, intensite_max_mm_h, mois, saison]`

---

### n5 — Modélisation et reconstitution 1980–2003
**Entrée :** `data/tableau_entrainement.csv` + `data/pluviometre_clean.csv`

**Objectif :** apprendre la relation `pluie journalière → intensité max` sur 2004–présent,  
puis l'appliquer aux années 1980–2003 pour **reconstituer** les intensités manquantes.

**Ce que fait ce notebook :**

1. **Construction des features :**
   - `pluie_mm` — pluie journalière brute
   - `log_pluie` — transformation log (améliore la linéarité)
   - `mois_sin`, `mois_cos` — encodage cyclique du mois (saisonnalité)

2. **Comparaison de 3 modèles par validation croisée 5-fold :**
   | Modèle | Avantage |
   |---|---|
   | LinearRegression | Simple, interprétable |
   | RandomForestRegressor | Capture les non-linéarités |
   | GradientBoostingRegressor | Meilleure précision en général |

3. **Sélection** du meilleur modèle par **R² moyen** en validation croisée

4. **Reconstitution** : le modèle prédit `log(1 + intensite)` → retransformation `expm1`

5. **Fallback R² < 0.5** : le notebook documente l'avertissement et continue (la transformation log est déjà appliquée)

**Figures produites dans `figures/n5/` :**
| Fichier | Description |
|---|---|
| `fig1_comparaison_modeles.png` | Barplot R² et RMSE des 3 modèles (avec barres d'erreur) |
| `fig2_residus.png` | Résidus vs prédits + distribution des résidus + Q-Q plot |
| `fig3_serie_reconstituee.png` | Série 1980–présent (observé en bleu / reconstitué en orange) |
| `fig4_validation_croisee.png` | Courbe d'apprentissage (train vs validation R²) |

**Livrable CSV :**
- `data/serie_complete.csv` — colonnes : `[date, intensite_max_mm_h, source, pluie_mm]`  
  avec `source` = `'observé'` ou `'reconstitué'`

---

### n6 — Analyse statistique des extrêmes
**Entrée :** `data/serie_complete.csv`

**Ce que fait ce notebook :**
1. Extrait les **maxima annuels** d'intensité (mm/h)
2. Ajuste 3 lois statistiques :
   | Loi | Paramètres | Nb paramètres |
   |---|---|---|
   | GEV (généralisée) | shape, loc, scale | 3 |
   | Gumbel (GEV shape=0) | loc, scale | 2 |
   | Lognormale | shape, loc=0, scale | 2 |
3. Sélectionne la loi par critère **AIC minimal** (pénalise la complexité)
4. Calcule les **valeurs de retour** pour T = 2, 5, 10, 20, 50, 100 ans
5. Estime les **intervalles de confiance 95%** par Bootstrap (n=500 rééchantillonnages)

**Fallback GEV :** si la GEV ne converge pas, fallback automatique sur Gumbel.

**Figures produites dans `figures/n6/` :**
| Fichier | Description |
|---|---|
| `fig1_lois_ajustees.png` | PDF des 3 lois superposées à l'histogramme des maxima |
| `fig2_qqplot.png` | Q-Q plots pour chaque loi (positions de Gringorten) |
| `fig3_periodes_retour.png` | Courbes de valeurs de retour (axe T en log) |
| `fig4_intervalles_confiance.png` | Courbe médiane + IC 95% Bootstrap |

---

### n7 — Courbes IDF (Intensité-Durée-Fréquence)
**Entrée :** `data/pluviographe_clean.csv` + `data/serie_complete.csv`

**Ce que fait ce notebook :**

1. **Maxima annuels par durée** : pour chaque durée (5 → 1440 min), calcule le max annuel d'intensité
2. **Ajustement Gumbel** par durée → valeurs de retour pour chaque durée et chaque T
3. **Tableau IDF** : matrice `[durée × période de retour]` d'intensités (mm/h)
4. **Formule de Montana** : ajustement de `I = a × D^b` par moindres carrés non linéaires (`curve_fit`)
   - `I` = intensité (mm/h)
   - `D` = durée (min)
   - `a`, `b` = paramètres dépendant de T
5. **IC Bootstrap** sur les courbes IDF pour T=10 ans et T=100 ans

**Figures produites dans `figures/n7/` :**
| Fichier | Description |
|---|---|
| `fig1_courbes_idf.png` | Courbes IDF sur axes log-log pour T = 2, 5, 10, 20, 50, 100 ans |
| `fig2_parametres_montana.png` | Évolution de a(T) et b(T) en fonction de T |
| `fig3_idf_enveloppe.png` | Enveloppe IC 95% pour T=10 et T=100 ans |

**Livrables CSV :**
- `data/tableau_idf.csv` — matrice intensités [durée × T]
- `data/parametres_idf.csv` — paramètres Montana `[T, a, b, R²]`

---

## ⚙️ Conventions de code

### Bloc de configuration (début de chaque notebook)
```python
import os
NB = "nX"
FIG_DIR = f"figures/{NB}"
DATA_DIR = "Donnees"
os.makedirs(FIG_DIR, exist_ok=True)
```

### Sauvegarde des figures
```python
fig.savefig(f'{FIG_DIR}/nom_figure.png', dpi=150, bbox_inches='tight')
```

### Résolution robuste des fichiers sources
```python
candidates = [Path('Donnees/KARA PLUIE JOURNALIERE.xls'), ...]
FICHIER = next((p for p in candidates if p.exists()), None)
```

### Environnement
- Python ≥ 3.9 (Anaconda recommandé)
- Librairies : `pandas`, `numpy`, `matplotlib`, `seaborn`, `scipy`, `scikit-learn`

---

## 🚀 Comment exécuter le pipeline

### Prérequis
```bash
# Depuis le terminal, dans le répertoire Projet_stage/
~/anaconda3/bin/jupyter notebook
```

### Ordre d'exécution strict

```
n1 → n2 → n3 → n4 → n5 → n6 → n7
```

> ⚠️ **Important :** le répertoire de travail du noyau Jupyter doit être `/home/kevin/Projet_stage/`.  
> Si Jupyter est lancé depuis un autre dossier, tous les chemins relatifs échoueront.

### Vérification rapide
```python
# Cellule à ajouter dans n'importe quel notebook pour vérifier le répertoire
import os
print(os.getcwd())  # Doit afficher : /home/kevin/Projet_stage
```

---

## 🔧 Gestion des anomalies

| Situation | Comportement automatique |
|---|---|
| Colonne introuvable | Détection automatique par nom + fallback par position |
| Fichier avec espaces dans le nom | Résolution par liste de candidats (`Path.exists()`) |
| Période commune vide (<10 jours) | Avertissement + utilisation de toute la période pluviographe |
| R² < 0.5 sur n5 | Documentation de l'avertissement, continuation avec transformation log |
| GEV non convergée | Fallback automatique sur Gumbel |
| Figure non générée | Bloc `try/except` → erreur loggée, notebook continue |
| Durée avec < 3 années | Ignorée pour l'ajustement Montana |

---

## 📊 Choix méthodologiques défendables

### Nettoyage des données
- **IQR × 3** (et non IQR × 1.5) : seuil plus conservateur, adapté aux pluies extrêmes qui sont réelles et non des erreurs de mesure.
- **Imputation médiane mensuelle** (et non moyenne) : la médiane est robuste aux valeurs extrêmes, pertinente pour des distributions asymétriques comme les précipitations.

### Modélisation (n5)
- **3 modèles testés** par validation croisée 5-fold : assure une comparaison objective sans surapprentissage.
- **Encodage cyclique du mois** (`sin`/`cos`) : capture la continuité de la saisonnalité (décembre–janvier se suivent).
- **Modélisation en log** : la distribution des intensités est log-normale → modéliser `log(I)` améliore la normalité des résidus.

### Analyse des extrêmes (n6)
- **Loi GEV généralisée** : cas général qui inclut Gumbel (shape=0), Fréchet (shape>0) et Weibull (shape<0).
- **Critère AIC** : pénalise les lois à plus de paramètres, évite le surapprentissage.
- **Positions de Gringorten** `(i-0.44)/(n+0.12)` : formule recommandée pour les maxima annuels en hydrologie.
- **Bootstrap IC 95%** : méthode non-paramétrique, ne suppose pas de distribution sur les paramètres.

### Courbes IDF (n7)
- **Formule de Montana** `I = a × D^b` : standard international en hydrologie urbaine, facilement intégrable dans les outils de dimensionnement.
- **Ajustement Gumbel par durée** : cohérent avec la théorie des valeurs extrêmes, recommandé par la OMM.
- **Axes log-log** : permet de vérifier la loi de puissance (Montana) visuellement.

---

## 📁 Fichiers livrables pour le rapport

| Fichier | Usage |
|---|---|
| `data/parametres_idf.csv` | Tableau Montana prêt à insérer dans le rapport |
| `data/tableau_idf.csv` | Tableau IDF complet (intensités par durée et période) |
| `figures/n7/fig1_courbes_idf.png` | Figure principale pour le rapport |
| `figures/n6/fig3_periodes_retour.png` | Courbes de valeurs de retour |
| `figures/n6/fig4_intervalles_confiance.png` | IC 95% pour justifier l'incertitude |

---

*Document généré automatiquement — Projet stage IDF Kara, Togo — 2026*
