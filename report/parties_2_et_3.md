# Analyse de paniers avec Apriori et FP-Growth — Parties 2, 3 et Extensions Master

---

## Partie 2 — Extraction de règles avec FP-Growth

À partir des 19 942 transactions nettoyées issues de la Partie 1, les paniers sont reconstruits avec les descriptions textuelles des produits pour garantir la lisibilité immédiate des règles d'association. La matrice d'incidence one-hot contient 19 942 factures pour 3 951 articles distincts.

L'application de l'algorithme FP-Growth avec un support minimal fixé à **3 %** (`min_support = 0.03`, soit au minimum ~600 factures) extrait **140 itemsets fréquents** en environ 8 à 9 secondes.

### Top 10 des itemsets les plus fréquents

| Rang | Article / Itemset | Support | Nombre de factures |
|:---:|---|---:|---:|
| 1 | WHITE HANGING HEART T-LIGHT HOLDER | 11.33 % | 2 260 |
| 2 | JUMBO BAG RED RETROSPOT | 10.49 % | 2 092 |
| 3 | REGENCY CAKESTAND 3 TIER | 9.97 % | 1 988 |
| 4 | PARTY BUNTING | 8.45 % | 1 685 |
| 5 | LUNCH BAG RED RETROSPOT | 7.84 % | 1 563 |
| 6 | ASSORTED COLOUR BIRD ORNAMENT | 7.30 % | 1 456 |
| 7 | SET OF 3 CAKE TINS PANTRY DESIGN | 6.95 % | 1 386 |
| 8 | PACK OF 72 RETROSPOT CAKE CASES | 6.62 % | 1 320 |
| 9 | LUNCH BAG BLACK SKULL. | 6.38 % | 1 272 |
| 10 | NATURAL SLATE HEART CHALKBOARD | 6.26 % | 1 248 |

### Règles d'association générées (Confiance ≥ 50 %)

À partir de ces 140 itemsets, 13 règles d'association respectent le seuil de confiance de 50 %. Les 5 règles les plus remarquables et diversifiées par famille de produits sont détaillées ci-dessous :

| N° | Règle {Antécédent} ⇒ {Conséquent} | Support | Confiance | Lift |
|:---:|---|---:|---:|---:|
| **1** | {PINK REGENCY TEACUP AND SAUCER} ⇒ {GREEN REGENCY TEACUP AND SAUCER} | 3.17 % | 82.6 % | 16.24 |
| **2** | {GREEN REGENCY TEACUP AND SAUCER} ⇒ {ROSES REGENCY TEACUP AND SAUCER} | 3.85 % | 75.7 % | 14.15 |
| **3** | {ALARM CLOCK BAKELIKE GREEN} ⇒ {ALARM CLOCK BAKELIKE RED} | 3.21 % | 65.3 % | 12.39 |
| **4** | {JUMBO BAG PINK POLKADOT} ⇒ {JUMBO BAG RED RETROSPOT} | 4.14 % | 67.7 % | 6.46 |
| **5** | {LUNCH BAG PINK POLKADOT} ⇒ {LUNCH BAG RED RETROSPOT} | 3.04 % | 55.6 % | 7.09 |

### Interprétation métier et actions marketing

1. **Règle 1 — Service à thé Regency (Rose ⇒ Vert)**
   - *Interprétation :* 82,6 % des clients achetant la tasse rose achètent également la tasse verte assortie. Le Lift de 16,24 indique que la probabilité d'acheter la tasse verte est multipliée par 16 lorsque la rose est présente dans le panier.
   - *Action marketing :* Création d'un bundle « Duo Regency Teatime » avec une remise de 10 % sur le lot, et affichage immédiat en pop-up d'ajout au panier.

2. **Règle 2 — Service à thé Regency (Vert ⇒ Roses motif floral)**
   - *Interprétation :* Plus de 3 acheteurs sur 4 (75,7 %) de la tasse verte complètent leur ensemble avec la version florale Roses.
   - *Action marketing :* Mécanique de cross-selling « Complétez votre service de table » : 3 tasses achetées, la 4e à -50 %, et regroupement des références sur une même tête de gondole / catégorie web dédiée.

3. **Règle 3 — Réveils Rétro Vintage (Vert ⇒ Rouge)**
   - *Interprétation :* Deux tiers des acheteurs du réveil rétro vert prennent aussi le modèle rouge (confiance 65,3 %, lift 12,39). Cela correspond à des achats de décoration coordonnée (deux pièces) ou à des cadeaux multiples.
   - *Action marketing :* Suggestion « Fréquemment achetés ensemble » avec case à cocher pour ajouter le réveil rouge directement depuis la page du réveil vert.

4. **Règle 4 — Sacs de rangement grand format (Pois Roses ⇒ Pois Rouges)**
   - *Interprétation :* 67,7 % des acheteurs du sac géant rose commandent aussi le sac rouge (best-seller de la boutique avec 10,5 % de support).
   - *Action marketing :* Offre dégressive par volume : « 1 sac à 3,95 £, le duo Rose + Rouge à 6,90 £ ».

5. **Règle 5 — Sacs repas isothermes (Lunch Bag Rose ⇒ Lunch Bag Rouge)**
   - *Interprétation :* Plus d'un acheteur sur deux (55,6 %) associe ces deux modèles (lift de 7,09). Typique des achats pour fratries ou rentrée scolaire.
   - *Action marketing :* Mise en avant dans les newsletters de rentrée « Back to School / Repas nomades ».

---

## Partie 3 — Comparaison Apriori vs FP-Growth

Sur l'ensemble des 19 942 transactions avec les mêmes paramètres (`min_support = 0.03`, `min_confidence = 0.5`), nous avons exécuté et chronométré les deux algorithmes :

| Critère | Apriori (`mlxtend`) | FP-Growth (`mlxtend`) |
|---|:---:|:---:|
| **Itemsets fréquents extraits** | 140 | 140 (strictement identiques) |
| **Règles d'association (conf ≥ 50 %)** | 13 | 13 (strictement identiques) |
| **Structure de données** | Tableaux / Génération de candidats | Arbre préfixé compact (**FP-Tree**) |
| **Passes sur la base de données** | $k$ passes (une par niveau) | 2 passes seulement |
| **Scalabilité à bas support** | Explosion combinatoire | Robuste et maintenable |

### Recommandations et synthèse critique

- **Algorithme recommandé :** **FP-Growth** est largement supérieur en environnement de production dès que le catalogue produit est étendu ou que le support minimal doit être abaissé pour détecter des associations rares.
- **Limites fondamentales :**
  1. *Corrélation vs Causalité :* Une co-occurrence statistique n'implique pas que l'achat de A déclenche l'achat de B (ex. : deux articles achetés en décembre en raison de Noël).
  2. *Biais de popularité :* Les produits très fréquents dominent les règles sans apporter de personnalisation fine.
- **Pistes d'amélioration :**
  1. *Mining temporel :* Calculer les règles sur des fenêtres glissantes (ex. : 30 jours) pour s'adapter à la saisonnalité.
  2. *Utility Mining :* Pondérer le support par la marge brute dégagée pour maximiser la rentabilité plutôt que le simple volume.

---

## Extensions « Version Master »

### Extension 1 — Analyse de sensibilité du support minimal

Nous avons fait varier `min_support` de 1 % à 5 % :

| Support minimal | Factures requises | Nb Itemsets fréquents | Nb Règles (conf ≥ 50 %) | Temps de calcul |
|:---:|:---:|:---:|:---:|:---:|
| **1 % (0.01)** | 199 | **1 810** | **814** | ~31 s |
| **2 % (0.02)** | 398 | **376** | **60** | ~16 s |
| **3 % (0.03)** | 598 | **140** | **13** | ~8 s |
| **5 % (0.05)** | 997 | **32** | **0** | ~4 s |

- **Analyse :** 
  - À 1 %, le volume explose (1 810 itemsets, 814 règles) : on capture des combinaisons rares mais le niveau de bruit statistique et de redondance est élevé.
  - À 5 %, aucun itemset de taille ≥ 2 n'atteint une confiance de 50 % : l'information est totalement appauvrie.
  - Le palier 2 % à 3 % constitue la zone de compromis optimale pour le e-commerce.

*(Les graphiques correspondants sont enregistrés dans `figures/analyse_sensibilite.png` et `figures/comparaison_temps.png`)*.

### Extension 2 — Segmentation géographique : Royaume-Uni vs International

Le partitionnement des transactions montre une dichotomie marquée entre le marché domestique britannique (90,6 % des transactions) et le marché export (9,4 %) :

- **Segment Royaume-Uni (18 069 transactions) :**
  - Règles phares : Objets d'ambiance et décorations festives `{WOODEN STAR CHRISTMAS SCANDINAVIAN} ⇒ {WOODEN HEART CHRISTMAS SCANDINAVIAN}` (Lift 27,27) et gamme Regency (Lift > 18).
  - *Comportement :* Achats pour usage personnel et décoration d'intérieur au coup de cœur.
  - *Recommandation :* Campagnes thématiques saisonnières et programmes de fidélité individuels.

- **Segment Hors-Royaume-Uni (1 873 transactions) :**
  - Règles phares : Articles d'événements, vaisselle jetable et univers enfants achetés en sets coordonnés complets : `{DOLLY GIRL CHILDRENS CUP, SPACEBOY CHILDRENS CUP} ⇒ {DOLLY GIRL CHILDRENS BOWL}` (Lift 35,34) et `{PACK OF 6 SKULL PAPER PLATES} ⇒ {PACK OF 20 SKULL PAPER NAPKINS}` (Lift 32,95).
  - *Comportement :* Revendeurs, commerçants locaux ou organisateurs de fêtes commandant des assortiments complets pour optimiser les coûts logistiques transfrontaliers.
  - *Recommandation :* Offres packagées B2B (« Party Pack Clé-en-main »), remises volumiques et seuils de franco de port attractifs.

### Extension 3 — Système de recommandation en production (FBT)

- **Architecture :** Pipeline batch nocturne calculant les règles FP-Growth sur 90 jours glissants, filtrant les règles à Lift ≥ 1.5, et injectant les paires recommandées dans un cache **Redis** en mémoire pour un service d'inférence sous les 10 ms.
- **Métriques à monitorer :**
  - *Offline :* Précision@K, Rappel@K, Couverture du catalogue.
  - *Online :* CTR sur les widgets FBT, Taux d'ajout au panier.
  - *Business :* Taux de conversion global et Panier Moyen (Average Order Value - AOV).
- **Protocole A/B Test :** Test randomisé 50/50 sur 3 semaines comparant le module FP-Growth (Groupe B) au module Top-Sellers non personnalisé (Groupe A), avec test bilatéral de significativité ($p < 0.05$).

### Extension 4 — Discussion critique et modèles hybrides

- **Limites majeures :** Absence d'historique personnel de l'utilisateur, insensibilité à la marge nette, risque d'enfermement dans une « bulle de popularité » écrasant la diversité du catalogue.
- **Solution industrielle pérenne :** Système hybride associant les règles d'association (pour le cross-selling direct en bas de page produit), la factorisation de matrice (filtrage collaboratif pour les affinités utilisateurs profondes) et les réseaux séquentiels (BERT4Rec / SASRec pour modéliser le parcours de navigation).

