# 🛡️ Topological Defense: Securing ANNs against PGD Attacks

![Python](https://img.shields.io/badge/Python-3.8%2B-blue)
![PyTorch](https://img.shields.io/badge/PyTorch-Red)
![Scikit-Learn](https://img.shields.io/badge/Scikit--Learn-Orange)
![Status](https://img.shields.io/badge/Status-Completed-success)

> **Détection d'attaques adversariales par analyse géométrique des activations internes d'un réseau de neurones.**

Ce projet a été développé dans le cadre d'un stage de recherche à l'Université Polytechnique de Catalogne. Il propose un système de défense externe et non-intrusif pour sécuriser un réseau de neurones (ANN) de détection de fraude bancaire face à des attaques cyber-adversariales par injection de gradient (PGD).

## 🧠 Le Concept

Les modèles de Deep Learning sont structurellement vulnérables aux attaques **Projected Gradient Descent (PGD)**. Un attaquant peut générer un bruit microscopique (Epsilon) sur une transaction frauduleuse pour aveugler l'IA et la forcer à prédire que la transaction est saine (Attack Success Rate de 100%).

**Notre solution :** Plutôt que de ré-entraîner l'ANN (coûteux et dégrade les performances), nous figeons le modèle et analysons sa **« surchauffe interne »**. 
1. Extraction des activations latentes des couches cachées.
2. Conversion en un graphe topologique ($3K$-D signatures) via **Matrice RBF, Mutual k-NN et PageRank**.
3. Détection de l'effondrement de la structure géométrique par un comité d'experts **One-Class SVM**.

## 📊 Résultats Clés (Dataset : Kaggle Credit Card Fraud)

* **Robustesse Furtive ($\epsilon = 1.0$) :** Interception de 91.84 % des fraudes camouflées.
* **Étanchéité Massive ($\epsilon \ge 2.0$) :** Taux d'interception parfait de **100.00 %** (l'ANN nu était à 0%).
* **Viabilité Métier :** Taux de Faux Positifs (FPR) stabilisé à **2.80 %**.
* **F1-Score Global :** **93.75 %** sur un audit multi-spectre.

## ⚙️ Architecture du Code

* `FraudDetection.py` / `notebook.ipynb` : Pipeline complet (Prétraitement, ANN, Attaque PGD, Extraction Graphes).
* **Parallélisation :** L'Ultra Grid-Search topologique est optimisé via `joblib` pour exploiter 100% des cœurs CPU, réduisant le temps d'inférence à moins de 20s par échelon de menace.

## 🚀 Installation & Utilisation

1. **Cloner le repository :**
   ```bash
   git clone [https://github.com/TON_NOM_UTILISATEUR/nom-du-repo.git](https://github.com/TON_NOM_UTILISATEUR/nom-du-repo.git)
   cd nom-du-repo

   pip install -r requirements.txt