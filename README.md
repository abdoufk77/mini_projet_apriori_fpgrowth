# Mini-projet : Analyse de paniers et recommandations produits

**Algorithmes Apriori et FP-Growth sur données réelles**

---

## 1. Contexte

Une enseigne de e-commerce nous donne ses données de ventes. Chaque ligne du fichier correspond à **un produit vendu dans une facture** (une transaction).

Quand on regarde les paniers d'achat, on remarque que certains produits reviennent souvent ensemble. C'est comme au supermarché : quelqu'un qui achète des pâtes prend souvent aussi de la sauce tomate.

Le but du projet est de **trouver automatiquement ces associations** sous forme de règles :

> **SI un client achète X, ALORS il achète souvent Y**

Ces règles permettent de proposer des **recommandations produits** : ventes croisées (cross-selling), offres groupées (bundles), placement des produits en rayon, suggestions « Vous aimerez aussi… ».

## 2. Objectif

À la fin du projet, on doit être capable de :

1. Préparer des données de ventes brutes pour l'analyse de paniers.
2. Appliquer **Apriori** et **FP-Growth** pour trouver les produits souvent achetés ensemble.
3. Interpréter les trois mesures : **support, confiance, lift**.
4. Transformer les règles en **recommandations concrètes** pour l'enseigne.
5. **Comparer** Apriori et FP-Growth (temps de calcul, nombre de règles, lisibilité).

## 3. Notions clés

| Mesure        | Question à laquelle elle répond                                     | Exemple                                                                            |
| ------------- | ------------------------------------------------------------------- | ---------------------------------------------------------------------------------- |
| **Support**   | Cette combinaison est-elle fréquente dans l'ensemble des factures ? | Support = 5 % : 5 factures sur 100 contiennent A et B.                             |
| **Confiance** | Parmi ceux qui achètent A, quelle part achète aussi B ?             | Confiance = 60 % : 6 acheteurs de A sur 10 prennent aussi B.                       |
| **Lift**      | A et B sont-ils vraiment liés, ou est-ce le hasard ?                | Lift > 1 : association réelle. Lift = 1 : indépendants. Lift < 1 : plutôt opposés. |

Une bonne règle a un support ni trop faible ni trop élevé, une confiance élevée et un lift supérieur à 1 (idéalement > 1,2).

## 4. Données utilisées

| Fichier                   | Taille                                     | Utilisation                        |
| ------------------------- | ------------------------------------------ | ---------------------------------- |
| `order_data.csv`          | Petit (quelques dizaines de transactions)  | Partie 0 : échauffement            |
| `transaction_dataset.csv` | Grand (plusieurs milliers de transactions) | Parties 1, 2 et 3 : analyse réelle |

> **À vérifier avant de commencer :** `transaction_dataset.csv` doit contenir au minimum un **identifiant de facture** et un **identifiant ou nom de produit**. Les colonnes pays et date sont utiles pour la segmentation. Si ce n'est pas le cas, il faut utiliser `Online Retail.xlsx` à la place.

## 5. Organisation du travail

| Parties                  | Responsable | Contenu                                                      | Livre à l'autre            |
| ------------------------ | ----------- | ------------------------------------------------------------ | -------------------------- |
| **Partie 0 et Partie 1** | _Membre A_  | Échauffement Apriori + préparation des données réelles       | Les transactions propres   |
| **Partie 2 et Partie 3** | _Membre B_  | FP-Growth, règles, comparaison Apriori/FP-Growth, extensions | Les résultats et l'analyse |

**Point de passage entre les deux (important) :** à la fin de la Partie 1, le Membre A fournit :

- la liste `transactions` (liste de listes, une liste de produits par facture) ;
- un fichier nettoyé `data/processed/transactions_clean.csv` qui conserve les colonnes **pays** et **date**, nécessaires à la segmentation.

Le Membre B part de ces deux éléments pour la Partie 2.

## 6. Détail des parties

### Partie 0 : mise en route (Membre A)

_On s'entraîne sur un petit fichier pour comprendre Apriori et les métriques._

1. Charger `order_data.csv`.
2. Construire une liste de listes : une liste d'articles par transaction.
3. Appliquer Apriori (bibliothèque `apyori`) avec `min_support = 0.25`, `min_confidence = 0.2`, `min_lift = 2`, `min_length = 2`.
4. Afficher 3 à 5 règles sous la forme `{X} ⇒ {Y}` et interpréter au moins 2 d'entre elles en langage simple.

### Partie 1 : préparation des données réelles (Membre A)

_Des données brutes sont presque toujours sales : il faut les nettoyer avant l'analyse._

1. Garder les colonnes utiles (facture, produit, quantité, pays, date).
2. Nettoyer : valeurs manquantes, quantités négatives, factures annulées (numéro commençant par « C »), lignes incohérentes.
3. Construire les transactions : **1 facture = 1 transaction = liste des produits achetés**.
4. Indiquer dans le rapport : le nombre de transactions, le nombre d'articles différents et 3 exemples de transactions.

### Partie 2 : extraction de règles avec FP-Growth (Membre B)

_On cherche les produits achetés ensemble sur les vraies données._

1. Encoder les transactions en tableau booléen (one-hot) avec `TransactionEncoder` (`mlxtend`).
2. Appliquer **FP-Growth** avec un `min_support` raisonnable (0.02 ou 0.03) et lister les 10 itemsets les plus fréquents.
3. Générer les règles avec `association_rules` (`metric="confidence"`, `min_threshold` entre 0.5 et 0.6) et les trier par confiance ou par lift.
4. Choisir **5 règles intéressantes**. Pour chacune :
   - la réécrire en phrase (« Les clients qui achètent A et B achètent souvent C ») ;
   - expliquer son support, sa confiance et son lift ;
   - proposer une action marketing (bundle, promotion, rangement en magasin, recommandation en ligne).

### Partie 3 : comparaison Apriori vs FP-Growth (Membre B)

_Quel algorithme est le meilleur pour ce type de données ?_

1. Appliquer Apriori sur le même dataset (ou un sous-échantillon) avec des paramètres compatibles.
2. Comparer :
   - le **temps de calcul** (avec `%time`) ;
   - le **nombre d'itemsets et de règles** ;
   - la **lisibilité** (trop de règles, règles redondantes).
3. Conclure : quel algorithme recommander et pourquoi, quelles sont les limites des règles d'association, et deux pistes d'amélioration.

## 7. Extensions « version Master » (obligatoires, Membre B)

Elles s'ajoutent aux Parties 2 et 3.

1. **Analyse de sensibilité** : faire varier `min_support` (0.01, 0.02, 0.03, 0.05) et observer le nombre d'itemsets, le nombre de règles et le type de règles (triviales ou plus rares et informatives). Présenter un tableau ou un graphique commenté.
2. **Segmentation** : choisir un critère (UK vs autres pays, haute vs basse saison, catégorie de produits). Pour au moins 2 segments, refaire l'extraction et sélectionner 3 à 5 règles. Comparer les règles communes, les règles spécifiques et les recommandations par segment.
3. **Lien avec les systèmes de recommandation** (~1 page) : intégration dans un module « Frequently bought together », métriques à suivre (taux de clic, taux de conversion, panier moyen), protocole de test (A/B testing, métriques offline).
4. **Discussion critique** :
   - limites : corrélation ≠ causalité, données manquantes, saisonnalité ;
   - risques : sur-personnalisation, sur-exposition de certains produits, perte de diversité des ventes ;
   - couplage possible avec d'autres approches (filtrage collaboratif, modèles séquentiels).

## 8. Livrables

1. **Un notebook Jupyter (`.ipynb`)** avec le code de préparation, Apriori, FP-Growth, l'extraction des règles, quelques graphiques et des cellules Markdown qui expliquent les choix et les résultats.
2. **Un rapport PDF de 3 à 4 pages** avec : introduction, méthodologie, résultats (tableaux de règles + interprétation), recommandations métier et conclusion avec la comparaison Apriori / FP-Growth.

## 9. Structure du dossier

```
mini_projet_apriori_fpgrowth/
│
├── .venv/                              # environnement virtuel (local, non partagé)
│
├── data/
│   ├── order_data.csv
│   ├── transaction_dataset.csv
│   └── processed/
│       └── transactions_clean.csv      # produit par la Partie 1
│
├── notebooks/
│   ├── 00_partie0_apriori_demo.ipynb   # Partie 0
│   └── Mini_Projet_Apriori_FPGrowth.ipynb  # notebook final (livrable)
│
├── figures/                            # graphiques pour le rapport
│
├── report/
│   └── rapport_mini_projet.docx         # rapport final (livrable)
│
├── .gitignore                          # contient : .venv/
├── requirements.txt
└── README.md
```

## 10. Installation et environnement de travail (VS Code)

On travaille dans **VS Code** avec un **environnement virtuel** : il garde les bibliothèques du projet séparées du reste de Python et garantit que les deux membres de l'équipe utilisent les mêmes versions.

### Prérequis

- **Python 3.9 ou plus récent** (python.org).
- **VS Code** avec deux extensions Microsoft : **Python** et **Jupyter** (`Ctrl+Shift+X`, puis recherche et installation). L'extension Jupyter permet d'ouvrir et d'exécuter les fichiers `.ipynb` directement dans VS Code.

### Étapes

**1. Ouvrir le dossier du projet** : `Fichier > Ouvrir un dossier` et choisir `mini_projet_apriori_fpgrowth`.

**2. Ouvrir un terminal** dans VS Code : `Terminal > Nouveau terminal`.

**3. Créer l'environnement virtuel** à la racine du projet :

```bash
python -m venv .venv
```

Sur Mac/Linux : `python3 -m venv .venv`.

**4. Activer l'environnement** :

| Système            | Commande                     |
| ------------------ | ---------------------------- |
| Windows PowerShell | `.venv\Scripts\Activate.ps1` |
| Windows CMD        | `.venv\Scripts\activate.bat` |
| Mac / Linux        | `source .venv/bin/activate`  |

Quand il est actif, `(.venv)` apparaît au début de la ligne du terminal.

**5. Installer les bibliothèques** :

```bash
pip install -r requirements.txt
```

**6. Choisir l'environnement dans le notebook** : ouvrir un fichier `.ipynb`, cliquer sur **Sélectionner le noyau** (en haut à droite), puis **Environnements Python**, et choisir celui qui contient `.venv`. Si VS Code propose d'installer `ipykernel`, accepter. Si l'environnement n'apparaît pas : `Ctrl+Shift+P`, puis **Python: Select Interpreter**.

**7. Exécuter les cellules** : `Maj+Entrée` pour une cellule, ou **Tout exécuter** (_Run All_) dans la barre du haut.

### Bibliothèques utilisées (`requirements.txt`)

| Bibliothèque            | Rôle                                                                  |
| ----------------------- | --------------------------------------------------------------------- |
| `pandas`, `numpy`       | Manipulation des données                                              |
| `matplotlib`, `seaborn` | Graphiques (top des produits, sensibilité du support)                 |
| `apyori`                | Apriori (Partie 0)                                                    |
| `mlxtend`               | `TransactionEncoder`, Apriori, FP-Growth, `association_rules`         |
| `openpyxl`              | Lecture de `Online Retail.xlsx` (seulement si ce fichier est utilisé) |
| `ipykernel`             | Exécution des notebooks dans VS Code                                  |

### Problèmes fréquents

- **Windows, PowerShell : « l'exécution de scripts est désactivée »** : lancer une fois cette commande, puis refaire l'étape 4 :

  ```powershell
  Set-ExecutionPolicy -Scope CurrentUser RemoteSigned
  ```

- **Fichier introuvable dans le notebook** : VS Code exécute le notebook depuis son propre dossier. Un notebook placé dans `notebooks/` doit donc lire les données avec le chemin `../data/order_data.csv`, à condition de respecter la structure du dossier ci-dessus.
- **Cellule `!pip install apyori`** : inutile si `requirements.txt` est déjà installé. Sinon, utiliser `%pip install apyori`, qui installe dans le bon environnement.
