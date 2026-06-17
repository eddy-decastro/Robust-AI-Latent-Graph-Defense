Détection de Fraude Bancaire via Graphes Latents et Robustesse Adversariale 🛡️🧠

Ce dépôt présente un pipeline de recherche et de développement avancé pour la détection d'anomalies (fraude bancaire), combinant l'apprentissage profond (Deep Learning) et l'apprentissage sur graphes (Graph Machine Learning). L'objectif principal est de concevoir un système de défense robuste face aux attaques adversariales de type White-Box (PGD), capables de tromper les réseaux de neurones classiques.

Plutôt que de se fier uniquement aux probabilités de sortie du réseau (vulnérables aux perturbations), ce projet innove en modélisant les activations internes des neurones sous forme de graphes topologiques. Un "Comité d'Experts" (modèles One-Class) analyse ces graphes pour isoler les signatures géométriques typiques des attaques.

📌 Sommaire

Contexte et Problématique

Architecture du Système

Méthodologie Détaillée

1. Prétraitement et Réseau Cible (Baseline)

2. Génération d'Attaques Adversariales (PGD)

3. Modélisation Topologique et Extraction de Signatures

4. Comité d'Experts (Détection d'Anomalies)

Résultats et Performances

Résultats de la Baseline (ANN)

Résultats du Système de Défense Topologique

Reproductibilité et Exécution

🎯 Contexte et Problématique

Les réseaux de neurones artificiels (ANN) sont performants pour détecter la fraude bancaire, mais ils présentent une faille majeure : ils sont vulnérables aux attaques adversariales. Un attaquant connaissant les poids du réseau (attaque "White-Box") peut ajouter une perturbation mathématique imperceptible aux données d'entrée (transactions) pour forcer le réseau à classer une fraude évidente comme une transaction "légitime".

Ce projet démontre :

La vulnérabilité d'un ANN standard face à des attaques PGD (Projected Gradient Descent).

La résilience d'une approche hybride qui intercepte les distorsions topologiques créées par l'attaque au sein même des couches cachées du réseau.

🧠 Architecture du Système

Le pipeline complet repose sur deux composants principaux fonctionnant en tandem :

Le Réseau Cible (ANN) : Un perceptron multicouche classique entraîné pour classer les transactions. Il est modifié via des hooks pour exposer ses activations internes.

Le Bouclier Topologique (Comité d'Experts) : Un ensemble d'algorithmes non-supervisés (One-Class SVM, Isolation Forest) qui n'analysent pas les données d'entrée, mais la forme (topologie) des activations du réseau cible.

🧪 Méthodologie Détaillée

1. Prétraitement et Réseau Cible (Baseline)

Données : Jeu de données creditcard.csv (déséquilibré : 99.8% sains / 0.17% fraudes).

Prétraitement : Suppression de la colonne temporelle, standardisation robuste (RobustScaler) de la colonne 'Amount', et séparation stratifiée (80% train / 20% val).

Architecture du Modèle : Réseau de neurones profond à 3 couches cachées (128 -> 64 -> 32) avec BatchNorm, Dropout et activation ReLU.

Entraînement : Utilisation de BCEWithLogitsLoss avec un poids (pos_weight) pour gérer le déséquilibre massif des classes. Les poids optimaux sont sauvegardés (.pth).

Reproductibilité : Verrouillage strict de toutes les graines (seeds) aléatoires (NumPy, PyTorch, CUDA).

2. Génération d'Attaques Adversariales (PGD)

Pour évaluer la robustesse du système, nous générons des attaques PGD (Projected Gradient Descent) sur l'espace des logits (sorties brutes).

Objectif de l'attaque : Modifier des transactions frauduleuses avérées pour que l'ANN les classe comme "saines".

Scan des Epsilons : L'intensité de la perturbation (Epsilon) varie de 0.1 (furtif) à 3.0 (massif). L'Attack Success Rate (ASR) mesure le pourcentage de fraudes réussissant à tromper l'ANN.

3. Modélisation Topologique et Extraction de Signatures

C'est le cœur de l'innovation de ce projet. Plutôt que d'analyser l'entrée perturbée, nous analysons la réaction interne du réseau cible.

Isolation des Nœuds Critiques : Calcul de la déviation absolue moyenne des activations latentes pour isoler les neurones (les dimensions) les plus sensibles à l'attaque PGD (Top-K nœuds).

Réseau Géométrique : Sur ces nœuds critiques, nous construisons des graphes de similarité :

Matrice de similarité via un noyau RBF (Radial Basis Function).

Filtrage des connexions parasites via une approche Mutual k-NN (k plus proches voisins mutuels).

Signatures 3K-D : Pour chaque observation, nous extrayons un vecteur de caractéristiques décrivant la topologie du graphe :

Magnitude : L'activation brute.

Degré : La densité de connexion locale.

PageRank : La centralité globale dans le graphe latent.

4. Comité d'Experts (Détection d'Anomalies)

Les signatures topologiques (3K-D) sont utilisées pour entraîner un Comité de modèles One-Class SVM (OCSVM).

Objectif : Apprendre ce qu'est une "signature topologique normale" (sur des transactions saines) pour détecter toute déviation causée par une attaque (anomalie).

Ultra Grid-Search Parallélisé : Optimisation exhaustive des hyperparamètres (K, k_nn, gamma, nu) sur plusieurs cœurs CPU (via joblib) pour maximiser l'interception tout en contrôlant les Faux Positifs.

Inférence par Consensus : Une transaction est bloquée uniquement si une majorité d'experts du comité identifie une anomalie structurelle.

📊 Résultats et Performances

Résultats de la Baseline (ANN)

Performance du réseau cible avant attaque (sur le set de validation) :

Accuracy: 99.95 %

Precision (Fraudes): 86.81 %

Recall (Fraudes): 80.61 %

F1-Score: 83.60 %

FPR (Faux Positifs): 0.02 %

Résultats du Système de Défense Topologique

Le "Bouclier" (Comité d'Experts) a été soumis à un audit statistique "Multi-Spectre" incluant des clients légitimes, des fraudes naturelles et des attaques PGD agressives.

Faux Positifs Contrôlés : Seulement 4.80% de FPR sur les citoyens honnêtes, garantissant la viabilité commerciale du système.

Interception des Fraudes Naturelles : Le comité bloque avec succès 86% (43/50) des fraudes classiques n'ayant subi aucune altération adversariale.

Résilience Absolue aux Attaques (Epsilon max 3.0) : Interception de 100% (50/50) des attaques de forte intensité ayant totalement trompé le réseau initial.

Analyse de la Frontière de Furtivité (Micro-Perturbations)

L'étude des micro-perturbations révèle la redoutable efficacité de l'approche topologique en comparant l'Attack Success Rate (ASR) du Hacker face à l'ANN nu, avec la capacité d'interception de notre Comité :

Epsilon 0.1 : ASR Hacker (ANN nu) = 0.00 % ➔ Interception par le Comité = 91.84 %

Epsilon 0.2 : ASR Hacker (ANN nu) = 0.00 % ➔ Interception par le Comité = 91.84 %

Epsilon 0.3 : ASR Hacker (ANN nu) = 6.12 % ➔ Interception par le Comité = 91.84 %

Epsilon 0.4 : ASR Hacker (ANN nu) = 22.45 % ➔ Interception par le Comité = 91.84 %

Epsilon 0.5 : ASR Hacker (ANN nu) = 30.61 % ➔ Interception par le Comité = 91.84 %

Epsilon 1.0 : ASR Hacker (ANN nu) = 100.00 % ➔ Interception par le Comité = 100.00 %

Bilan : Même face à une perturbation ultra-furtive (Epsilon = 0.1), incapable de tromper l'ANN (ASR = 0%), le gradient modifie l'espace latent. Le Comité topologique détecte la déformation géométrique et bloque l'attaque avec une précision supérieure à 91% avant même qu'elle ne devienne une menace concrète.

🚀 Reproductibilité et Exécution

Cloner le dépôt

git clone [https://github.com/eddy-decastro/Robust-AI-Latent-Graph-Defense.git](https://github.com/eddy-decastro/Robust-AI-Latent-Graph-Defense.git)
cd Robust-AI-Latent-Graph-Defense


Environnement : Installez les dépendances :

pip install torch scikit-learn pandas numpy matplotlib seaborn joblib


Données : Téléchargez le dataset creditcard.csv (Kaggle) et placez-le dans le répertoire du projet (vérifiez le file_path dans le script de prétraitement).

Exécution : Exécutez les blocs de code de manière séquentielle, de la configuration initiale jusqu'aux audits statistiques et à la génération de graphiques, en utilisant les notebooks ou scripts fournis.