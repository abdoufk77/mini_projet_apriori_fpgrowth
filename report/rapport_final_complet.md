# Rapport Final — Mini-Projet : Analyse de Paniers et Recommandations Produits
**Algorithmes Apriori et FP-Growth sur Données Réelles de Ventes**

---

## 1. Introduction et Contexte

Dans le secteur du commerce électronique et de la grande distribution, la compréhension des comportements d'achat est un levier stratégique majeur pour optimiser le merchandising, les ventes croisées (*cross-selling*), les offres groupées (*bundles*) et l'agencement du catalogue.

Ce projet a pour objectif d'extraire automatiquement des régularités d'achat sous la forme de **règles d'association** de type :
$$\{X\} \implies \{Y\}$$
signifiant : *« Si un client achète l'article ou l'ensemble d'articles $X$, alors il achète très fréquemment l'article $Y$. »*

Nous mobilisons pour cela les trois mesures fondamentales de l'analyse de paniers :
- **Support :** Fréquence relative de l'ensemble d'articles dans l'ensemble des transactions.
- **Confiance :** Probabilité conditionnelle d'observer le conséquent $Y$ sachant que l'antécédent $X$ est présent dans le panier.
- **Lift :** Ratio mesurant la force de la dépendance statistique par rapport à l'indépendance aléatoire ($\text{Lift} > 1$ indique une corrélation positive avérée).

---

## 2. Partie 0 — Mise en route avec Apriori (Données de test)

Un premier entraînement a été conduit sur un jeu d'échauffement synthétique (`order_data.csv`) comportant 20 paniers d'épicerie. Les seuils appliqués étaient : $\text{min\_support} = 25\,\%$, $\text{min\_confidence} = 20\,\%$ et $\text{min\_lift} = 2$.

| Règle extraite | Support | Confiance | Lift |
|---|---:|---:|---:|
| {mouthwash} ⇒ {toothpaste} | 30.0 % | 85.7 % | 2.14 |
| {toothpaste} ⇒ {mouthwash} | 30.0 % | 75.0 % | 2.14 |
| {brush} ⇒ {toothpaste} | 25.0 % | 100.0 % | 2.50 |
| {toothpaste} ⇒ {brush} | 25.0 % | 62.5 % | 2.50 |
| {milk, shampoo} ⇒ {cereals} | 25.0 % | 100.0 % | 2.50 |

### Interprétation
- La règle `{brush} ⇒ {toothpaste}` possède une confiance parfaite de 100 % : la totalité des 5 paniers contenant une brosse à dents contiennent également du dentifrice. Le dentifrice apparaissant dans 40 % des paniers au global, le Lift est de $1.0 / 0.4 = 2.50$, prouvant une association forte.

---

## 3. Partie 1 — Préparation des données réelles de vente

Le jeu de données réel (`transaction_dataset.csv`) totalise **541 909 lignes de vente brutes**.

### Protocole de nettoyage
1. **Élimination des annulations :** Suppression des factures débutant par la lettre « C » (9 288 lignes).
2. **Valeurs manquantes :** Suppression des enregistrements présentant des champs nuls ou vides (7 422 lignes).
3. **Frais et régularisations comptables :** Exclusion des codes non produits (`POST`, `DOT`, `M`, `BANK CHARGES`, etc. — 2 322 lignes).
4. **Quantités non positives :** Retrait des quantités nulles ou négatives (462 lignes).
5. **Validation de cohérence :** Vérification de l'unicité du pays et de la date par numéro de facture.

### Volumétrie finale
Après application de ces filtres, **522 415 lignes de vente valides** sont conservées et sauvegardées dans `data/processed/transactions_clean.csv`. Elles correspondent à :
- **19 942 transactions uniques (factures)**
- **3 831 à 3 951 articles distincts**

---

## 4. Partie 2 — Extraction de règles avec FP-Growth

Pour assurer une lisibilité opérationnelle immédiate aux équipes métier, chaque facture a été agrégée en un panier de **descriptions textuelles** de produits. La matrice d'incidence booléenne (One-Hot) obtenue possède la dimension :
$$19\,942 \text{ transactions} \times 3\,951 \text{ articles}$$

### 4.1 Itemsets fréquents
En fixant un seuil de support minimal de **3 %** (`min_support = 0.03`, soit au minimum ~600 factures), l'algorithme FP-Growth identifie **140 itemsets fréquents** en environ 8 secondes.

Le Top 5 des articles les plus fréquents est constitué de :
1. `WHITE HANGING HEART T-LIGHT HOLDER` : Support = 11.33 % (2 260 factures)
2. `JUMBO BAG RED RETROSPOT` : Support = 10.49 % (2 092 factures)
3. `REGENCY CAKESTAND 3 TIER` : Support = 9.97 % (1 988 factures)
4. `PARTY BUNTING` : Support = 8.45 % (1 685 factures)
5. `LUNCH BAG RED RETROSPOT` : Support = 7.84 % (1 563 factures)

### 4.2 Règles d'association remarquables
Avec un seuil de confiance minimal de **50 %**, 13 règles d'association sont extraites. Cinq règles hautement significatives et couvrant différents rayons sont analysées :

| Règle {Antécédent} ⇒ {Conséquent} | Support | Confiance | Lift | Famille de produit |
|---|---:|---:|---:|---|
| **{PINK REGENCY TEACUP} ⇒ {GREEN REGENCY TEACUP}** | 3.17 % | 82.6 % | **16.24** | Art de la table vintage |
| **{GREEN REGENCY TEACUP} ⇒ {ROSES REGENCY TEACUP}** | 3.85 % | 75.7 % | **14.15** | Art de la table vintage |
| **{ALARM CLOCK GREEN} ⇒ {ALARM CLOCK RED}** | 3.21 % | 65.3 % | **12.39** | Décoration rétro |
| **{JUMBO BAG PINK POLKADOT} ⇒ {JUMBO BAG RED}** | 4.14 % | 67.7 % | **6.46** | Rangement / Cabas |
| **{LUNCH BAG PINK POLKADOT} ⇒ {LUNCH BAG RED}** | 3.04 % | 55.6 % | **7.09** | Repas nomade |

### 4.3 Recommandations marketing associées
1. **Vaisselle Regency (Lift 16.24 & 14.15) :** Création d'un bundle « Coffret Duo Regency » avec packaging cadeau et remise promotionnelle de 10 % pour encourager l'achat de la collection complète.
2. **Réveils rétro (Lift 12.39) :** Recommandation dynamique « Vous aimerez aussi » intégrée à la fiche article avec ajout groupé en 1 clic.
3. **Cabas et sacs isothermes (Lift 6.46 & 7.09) :** Remise sur volume (« 1 sac = 3.95 £, le duo = 6.90 £ ») et mise en avant dans les newsletters de rentrée scolaire.

---

## 5. Partie 3 — Comparaison expérimentale : Apriori vs FP-Growth

L'évaluation comparative a été menée sur les 19 942 transactions avec les mêmes hyperparamètres :

| Dimension d'évaluation | Apriori | FP-Growth |
|---|:---:|:---:|
| **Itemsets découverts** | 140 | 140 (strictement identiques) |
| **Règles générées** | 13 | 13 (strictement identiques) |
| **Paradigme de minage** | Génération de candidats niveau par niveau | Arbre préfixé compact (**FP-Tree**) |
| **Passes sur la base de données** | $k$ lectures complètes | 2 lectures seulement |
| **Passage à l'échelle (Scalabilité)** | Explosion exponentielle des candidats si support bas | Excellente conservation de mémoire et vitesse |

### Conclusion et préconisations
- **Recommandation :** L'algorithme **FP-Growth doit être privilégié en environnement de production**.
- **Limites conceptuelles :** La corrélation ne prouve pas la causalité, et les articles à très forte popularité écrasent la personnalisation.
- **Axes d'amélioration :** Découpage temporel glissant (fenêtre de 30 jours) et pondération par la marge unitaire (*Utility-based pattern mining*).

---

## 6. Extensions Master

### 6.1 Analyse de sensibilité du support minimal
L'exploration de `min_support` sur la grille $\{0.01, 0.02, 0.03, 0.05\}$ met en lumière :
- À **1 %** : 1 810 itemsets et 814 règles. L'espace de recherche capte des micro-associations de niche, mais au prix d'un bruit important.
- À **2 %** : 376 itemsets et 60 règles. Bon équilibre pour des recommandations transversales.
- À **3 %** : 140 itemsets et 13 règles. Ensemble ultra-pertinent pour des actions commerciales ciblées.
- À **5 %** : 32 itemsets et **0 règle** à confiance ≥ 50 %. Les articles très fréquents sont disséminés dans trop de paniers hétérogènes.

### 6.2 Segmentation géographique : Royaume-Uni vs International
- **Royaume-Uni (90.6 % des ventes) :** Dominance de décorations d'intérieur coup de cœur et articles de fêtes scandinaves (`{WOODEN STAR} ⇒ {WOODEN HEART}`, Lift = 27.27). Les clients sont majoritairement des particuliers.
- **International / Hors-UK (9.4 % des ventes) :** Prédominance d'achats groupés d'articles d'événements et de vaisselle pour enfants (`{DOLLY GIRL BOWL, SPACEBOY CUP} ⇒ {DOLLY GIRL CUP}`, Lift = 35.34 ; vaisselle jetable Halloween, Lift = 32.95). Les clients sont des grossistes, commerçants ou organisateurs d'événements cherchant à rentabiliser le fret international.

### 6.3 Déploiement en production (Système de recommandation)
- **Architecture :** Calcul batch régulier de FP-Growth sur 90 jours glissants, filtrage des règles par Lift $\ge 1.5$, stockage clé-valeur dans **Redis** pour un temps de réponse inférieur à 10 ms sur le front-end.
- **Métriques :** Offline (Précision@K, Rappel@K, Couverture catalogue) et Online (CTR, Taux d'ajout au panier, Panier moyen AOV).
- **A/B Testing :** Comparaison sur 3 semaines entre Groupe A (Top-sellers) et Groupe B (Règles FP-Growth contextuelles).

### 6.4 Discussion critique et modèle hybride
Les limites des règles d'association (manque d'historique utilisateur, bulle de popularité) invitent à concevoir un modèle hybride :
$$\text{Recommandation} = f(\text{FP-Growth}, \text{Filtrage Collaboratif ALS}, \text{Modèles Séquentiels BERT4Rec})$$
Cette synergie réunit la force du cross-selling contextuel direct et la profondeur des affinités personnelles à long terme.

