# Projet-ML
Objectif du Projet

L'objectif est de construire une machine capable de lire automatiquement des tweets (récoltés sur X) et de les trier selon leur rapport à la science. C'est un problème classique de Machine Learning (Apprentissage Supervisé) où l'on fournit à l'ordinateur des exemples déjà triés pour qu'il en déduise les règles logiques.

L'analogie filée tout au long de ce projet est celle d'un Centre de Tri Postal : les tweets sont nos lettres, et les algorithmes sont nos employés de tri.
Architecture et Découpage Logique

Le code (disponible dans le notebook Jupyter) est découpé en grandes étapes chronologiques :
Étape 1 : Réception et Nettoyage du Courrier (Pré-traitement)

Avant de lire le texte, il faut le rendre compréhensible pour la machine en éliminant le "bruit" visuel.

    Uniformisation : Passage de tout le texte en lettres minuscules.

    Désencombrement : Suppression des adresses internet (http...) et des mentions d'utilisateurs (@...) via des expressions régulières (Regex).

    Filtrage du vocabulaire : Suppression de la ponctuation, des caractères spéciaux et des "mots vides" (Stop Words en anglais, comme "the", "and", "is") grâce à la bibliothèque NLTK.

Étape 2 : Traduction Mathématique (Vectorisation)

Une machine ne sait pas lire des lettres de l'alphabet, elle ne calcule qu'avec des nombres.

    L'outil : Nous utilisons un TfidfVectorizer (limité aux 1000 mots les plus fréquents pour des raisons de performance).

    La logique : Il transforme chaque tweet en un grand tableau de chiffres (un code-barres). Il donne un poids mathématique fort aux mots rares et spécifiques (ex: gynecologist) et un poids faible aux mots trop communs.

Étape 3 : La Chaîne de Montage (Pipelines)

Pour éviter les erreurs de manipulation et regrouper le traducteur et l'algorithme, nous avons encapsulé notre processus dans un Pipeline Python de scikit-learn. Le texte brut entre d'un côté, la prédiction sort de l'autre.
Les 3 Tâches de Classification
Tâche 1 : Le Tri Principal ({SCI} vs {NON-SCI})

    Objectif : Séparer les tweets scientifiques du reste.

    Modèle utilisé : LogisticRegression (Régression Logistique).

    Stratégie face au déséquilibre : Le jeu de données contient beaucoup plus de tweets normaux que de tweets scientifiques. Nous avons utilisé le paramètre class_weight='balanced' pour forcer l'algorithme à accorder plus de valeur à la classe minoritaire, faisant ainsi bondir notre capacité de détection (le Rappel / Recall).

Tâche 2 : Le Sous-Tri Binaire ({CLAIM, REF} vs {CONTEXT})

    Objectif : Uniquement sur les tweets scientifiques, séparer les Affirmations/Références du simple Contexte.

    Défi rencontré : Le manque critique de données de "Contexte" (33 exemples seulement contre 342). Malgré l'équilibrage du modèle, la machine peine à trouver l'aiguille dans la botte de foin par manque de vocabulaire de référence.

Tâche 3 : Le Tri Fin ({CLAIM} vs {REF} vs {CONTEXT})
    Objectif : Classification à 3 classes sur les données scientifiques.
    Défi rencontré : Les données sont "multi-labels" dans la réalité (un chercheur affirme une chose tout en postant le lien de l'étude).
    Solution logique : Création d'une fonction de priorité (Si c'est un lien -> REF, sinon si c'est une affirmation -> CLAIM, sinon -> CONTEXT) pour forcer une classification mutuellement exclusive. L'analyse révèle que l'algorithme confond logiquement CLAIM et REF car leurs frontières sémantiques se chevauchent énormément.

Évaluation (Le Contrôle Qualité)
Nous avons délibérément évité de nous fier uniquement à l'Accuracy (le pourcentage global de réussite), car il est trompeur sur des données déséquilibrées.
Chaque tâche est évaluée à l'aide de :

    La Matrice de Confusion : Pour visualiser précisément quelles "lettres" ont été jetées dans les mauvais bacs.

    Le Rapport de Classification : Pour surveiller la Précision, le Rappel, et surtout le Macro F1-Score (la moyenne équilibrée).

    L'Analyse Qualitative : Une inspection visuelle des tweets mal classés pour comprendre les biais du modèle (ex: la suppression des URLs qui détruit la capacité à détecter une référence).
