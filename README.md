# Phishing-url-detection
# 🔐 Détection des URL de Phishing par Machine Learning

<p align="center">
  <img src="https://img.shields.io/badge/Python-3.10+-blue?style=for-the-badge&logo=python&logoColor=white"/>
  <img src="https://img.shields.io/badge/scikit--learn-ML-orange?style=for-the-badge&logo=scikit-learn&logoColor=white"/>
  <img src="https://img.shields.io/badge/Jupyter-Notebook-F37626?style=for-the-badge&logo=jupyter&logoColor=white"/>
  <img src="https://img.shields.io/badge/Cybersécurité-Phishing-red?style=for-the-badge&logo=shield&logoColor=white"/>
  <img src="https://img.shields.io/badge/Statut-Terminé-brightgreen?style=for-the-badge"/>
</p>

<p align="center">
  <b>Projet Machine Learning</b><br/>

</p>

---

## 📌 Table des Matières

- [À propos du projet](#-à-propos-du-projet)
- [Dataset](#-dataset)
- [Pipeline de traitement](#-pipeline-de-traitement)
- [Features sélectionnées](#-features-sélectionnées)
- [Modèles comparés](#-modèles-comparés)
- [Résultats détaillés](#-résultats-détaillés)
- [Meilleur modèle](#-meilleur-modèle--svm-rbf)
- [Structure du projet](#-structure-du-projet)
- [Bibliothèques utilisées](#-bibliothèques-utilisées)
- [Perspectives](#-perspectives)
- [Auteurs](#-auteurs)

---

## 🎯 À propos du projet

Le **phishing** est une technique de cyberattaque visant à usurper l'identité d'entités légitimes (banques, réseaux sociaux, administrations) via de fausses URLs, dans le but de voler les informations d'identification des utilisateurs.

Face à la multiplication et à la sophistication croissante de ces attaques, les méthodes de détection basées sur des règles manuelles montrent leurs limites. Ce projet propose une approche basée sur le **Machine Learning** pour détecter automatiquement les URLs malveillantes.

### Objectifs

- ✅ Construire un **classificateur binaire** capable de distinguer URLs légitimes et de phishing
- ✅ Extraire et sélectionner les **features les plus discriminantes** à partir des caractéristiques structurelles des URLs
- ✅ Comparer les performances de **4 algorithmes** de Machine Learning
- ✅ Identifier le modèle le plus adapté au contexte de la cybersécurité

---

## 📊 Dataset

| Propriété | Valeur |
|-----------|--------|
| **Source** | Dataset Phishing URLs |
| **Nombre d'URLs** | 11 430 |
| **Classes** | Légitime (0) / Phishing (1) |
| **Distribution** | 50 % / 50 % — parfaitement équilibré |
| **Features initiales** | 87 features + 1 colonne cible |
| **Features finales** | 20 features sélectionnées |
| **Valeurs manquantes** | Aucune |

```
df['status'].value_counts()
# 0 (Légitime) : 5715
# 1 (Phishing) : 5715
```

> ⚠️ Le fichier `dataset_phishing.csv` n'est pas inclus dans ce dépôt en raison de sa taille.  
> Vous pouvez le télécharger sur [Kaggle — Web Page Phishing Detection](https://www.kaggle.com/datasets/shashwatwork/web-page-phishing-detection-dataset).

---

## 🔧 Pipeline de traitement

```
Raw Data (11 430 URLs, 88 colonnes)
         │
         ▼
┌─────────────────────┐
│  1. Chargement EDA  │  ← ydata-profiling, statistiques descriptives
└─────────────────────┘
         │
         ▼
┌──────────────────────────┐
│  2. Détection Outliers   │  ← Méthode IQR + Clipping
└──────────────────────────┘
         │
         ▼
┌────────────────────────────┐
│  3. Feature Engineering    │  ← Encodage, nouvelles features URL
└────────────────────────────┘
         │
         ▼
┌─────────────────────────────────────────────────┐
│  4. Sélection de Features                       │
│     • Corrélation > 0.95 → drop                 │
│     • VarianceThreshold                         │
│     • SelectKBest (f_classif) → Top 20 features │
└─────────────────────────────────────────────────┘
         │
         ▼
┌───────────────────────────────────────┐
│  5. Normalisation (StandardScaler)    │  ← Fit sur Train uniquement
└───────────────────────────────────────┘
         │
         ▼
┌────────────────────────────────────────────────────┐
│  6. Modélisation & Comparaison                     │
│     KNN | SVM | Arbre de Décision | Naive Bayes    │
└────────────────────────────────────────────────────┘
         │
         ▼
   Meilleur modèle : SVM (Accuracy 95.80% | AUC 99.01%)
```

### Détail des étapes

#### 🔍 Exploration des données (EDA)
- Rapport automatique généré avec **ydata-profiling** (HTML interactif)
- Statistiques descriptives (min, max, mean, std)
- Visualisation des distributions et corrélations entre variables
- Identification des valeurs extrêmes

#### 🧹 Traitement des valeurs aberrantes
- Méthode **IQR** (Interquartile Range)
- **Borne inf** = Q1 − 1.5 × IQR | **Borne sup** = Q3 + 1.5 × IQR
- Traitement par **Clipping** (valeurs remplacées par les bornes, pas supprimées)
- Correction spécifique : `domain_age` négatif → 0 (valeur sentinelle = inconnu)

#### ⚙️ Feature Engineering
- Encodage de la cible : `légitime → 0`, `phishing → 1`
- Création de nouvelles features : `url_length`, `num_digits`, `num_special`, `https_flag`
- Normalisation : **StandardScaler** (mean=0, std=1) — fit sur train uniquement pour éviter le data leakage

---

## 🏆 Features sélectionnées

Les **20 features** retenues après sélection par SelectKBest (f_classif) :

| # | Feature | Description |
|---|---------|-------------|
| 01 | `length_url` | Longueur totale de l'URL |
| 02 | `length_hostname` | Longueur du nom de domaine |
| 03 | `nb_dots` | Nombre de points dans l'URL |
| 04 | `nb_slash` | Nombre de slashes |
| 05 | `nb_www` | Présence de "www" |
| 06 | `ratio_digits_url` | Ratio de chiffres dans l'URL |
| 07 | `length_words_raw` | Longueur des mots bruts |
| 08 | `longest_words_raw` | Mot le plus long dans l'URL |
| 09 | `longest_word_path` | Mot le plus long dans le chemin |
| 10 | `avg_word_path` | Longueur moyenne des mots du chemin |
| 11 | `nb_hyperlinks` | Nombre de liens hypertextes |
| 12 | `ratio_intHyperlinks` | Ratio de liens internes |
| 13 | `ratio_extErrors` | Ratio d'erreurs sur liens externes |
| 14 | `domain_registration_length` | Durée d'enregistrement du domaine |
| 15 | `domain_age` | Âge du domaine |
| 16 | `google_index` | Indexation par Google (0/1) |
| 17 | `page_rank` | Score PageRank du domaine |
| 18 | `url_length` | Longueur URL (feature créée) |
| 19 | `num_digits` | Nombre de chiffres (feature créée) |
| 20 | `num_special` | Nombre de caractères spéciaux (feature créée) |

---

## 🤖 Modèles comparés

### 1. K-Nearest Neighbors (KNN)

> Algorithme non paramétrique basé sur la similarité. Classe une URL en effectuant un vote majoritaire parmi ses k plus proches voisins dans l'espace des features.

- **Distance** : Euclidienne
- **Recherche du k optimal** : évaluation de k ∈ [1, 34] par F1-score
- **k optimal** : k = 15 → F1 = 0.9506

### 2. Support Vector Machine (SVM) ⭐

> Cherche l'hyperplan de séparation optimal qui maximise la marge entre les deux classes. Le kernel RBF permet de gérer des frontières non linéaires.

- **Kernel** : RBF (Radial Basis Function)
- **C** : 1.0 | **Gamma** : scale | **probability** : True

### 3. Arbre de Décision

> Construit des règles de classification interprétables de manière récursive. Seul modèle pleinement lisible par un expert métier.

- **max_depth** : 5 (pour limiter le surapprentissage)
- **Règles clés** : `google_index`, `page_rank`, `nb_hyperlinks`

### 4. Naive Bayes (GaussianNB)

> Applique le théorème de Bayes en supposant l'indépendance conditionnelle entre les features (hypothèse naïve).

- **Validation croisée 5-fold** : 83.65% ± 0.82%

---

## 📈 Résultats détaillés

| Modèle | Accuracy | Précision | Recall | F1-Score | ROC-AUC | Interprétable |
|--------|:--------:|:---------:|:------:|:--------:|:-------:|:-------------:|
| KNN (k=15) | 95.10% | 94.64% | 95.48% | 95.06% | 98.61% | ❌ |
| **SVM (RBF)** | **95.80%** | **95.43%** | **96.10%** | **95.76%** | **99.01%** | ❌ |
| Arbre (depth=5) | 92.39% | 91.45% | 93.53% | 92.38% | 96.43% | ✅ |
| Naive Bayes | 83.42% | 84.50% | 83.42% | 83.38% | 93.98% | ⚠️ |

### Matrice de confusion — SVM (meilleur modèle)

```
                  Prédit Légitime    Prédit Phishing
Réel Légitime         1105               52
Réel Phishing           44             1085
```

- **Vrais Positifs** : 1085 phishings correctement détectés
- **Faux Négatifs** : seulement 44 phishings manqués ✅

---

## 🥇 Meilleur modèle : SVM (RBF)

Le **SVM avec kernel RBF** est retenu comme modèle recommandé pour les raisons suivantes :

| Critère | Résultat |
|---------|----------|
| 🎯 Accuracy maximale | **95.80 %** sur le jeu de test |
| 📊 ROC-AUC record | **99.01 %** — discrimination quasi-parfaite |
| ⚖️ Équilibre Précision/Recall | 95.43 % / 96.10 % |
| 🛡️ Sécurité maximale | **FN = 44** seulement (très peu de phishing non détecté) |

---

## 📁 Structure du projet

```
phishing-url-detection/
│
├── 📓 Phishing_ML.ipynb              # Notebook principal (pipeline complet)
│
├── 📄 rapport/
│   └── Rapport_Phishing_ML.docx     # Rapport de projet complet
│
├── 📦 models/                        # Modèles sauvegardés (optionnel)
│   └── scaler.pkl                   # StandardScaler entraîné
│
├── 📊 data/
│   └── README_data.md               # Instructions pour télécharger le dataset
│
└── 📖 README.md                     # Ce fichier
```

---


### 1. Cloner le dépôt

```bash
git clone https://github.com/VOTRE_USERNAME/phishing-url-detection.git
cd phishing-url-detection
```

### 2. Installer les dépendances

```bash
pip install pandas numpy scikit-learn matplotlib seaborn ydata-profiling
```

### 3. Télécharger le dataset

Téléchargez `dataset_phishing.csv` depuis [Kaggle](https://www.kaggle.com/datasets/shashwatwork/web-page-phishing-detection-dataset) et placez-le à la racine du projet.

### 4. Lancer le notebook

```bash
jupyter notebook Phishing_ML.ipynb
```

---

## 📚 Bibliothèques utilisées

```python
pandas          # Manipulation des données
numpy           # Calcul numérique
scikit-learn    # Modèles ML, prétraitement, métriques
matplotlib      # Visualisation
seaborn         # Visualisation statistique
ydata-profiling # EDA automatisée
```

---

## 🔭 Perspectives

- 🧠 **Deep Learning (LSTM)** : Appliquer des réseaux récurrents directement sur la séquence de caractères des URLs
- ⚡ **Entraînement en temps réel** : Pipeline d'apprentissage continu alimenté par de nouvelles URLs détectées
- 🌐 **API REST / Extension Chrome** : Déployer le modèle pour une protection en temps réel de l'utilisateur
- 🔗 **Intégration SIEM** : Coupler le classificateur avec un système de gestion d'événements de sécurité
- 🌲 **Méthodes d'ensemble** : Tester Random Forest, XGBoost, LightGBM pour améliorer encore les performances

---


---

<p align="center">
  Projet réalisé dans le cadre du cours <b>Machine Learning en Sécurité Informatique</b>
</p>
