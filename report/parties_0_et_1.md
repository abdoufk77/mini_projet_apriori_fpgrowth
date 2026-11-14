# Analyse de paniers avec Apriori et FP-Growth

## Partie 0 — Mise en route avec Apriori

Le fichier d’échauffement contient 20 paniers. Le support minimal de 25 % impose la présence commune des articles dans au moins cinq paniers. Les autres seuils sont une confiance de 20 %, un lift de 2 et un itemset d’au moins deux articles.

| Règle | Support | Confiance | Lift |
| --- | ---: | ---: | ---: |
| {mouthwash} → {toothpaste} | 30% | 85.7% | 2.14 |
| {toothpaste} → {mouthwash} | 30% | 75.0% | 2.14 |
| {brush} → {toothpaste} | 25% | 100.0% | 2.50 |
| {toothpaste} → {brush} | 25% | 62.5% | 2.50 |
| {milk, shampoo} → {cereals} | 25% | 100.0% | 2.50 |

Parmi les 7 paniers contenant du bain de bouche, 6 contiennent aussi du dentifrice. Cela donne une confiance de 85,7 % et un support de 30 %. Le dentifrice seul apparaît dans 40 % des paniers, ce qui donne un lift de 2,14.

Les 5 paniers contenant une brosse contiennent tous du dentifrice. La règle a une confiance de 100 %, un support de 25 % et un lift de 2,50. Dans le sens inverse, 5 des 8 paniers contenant du dentifrice contiennent une brosse. La confiance est donc de 62,5 %.

## Partie 1 — Préparation des données réelles

Le fichier source contient 541 909 lignes de vente. Le nettoyage écarte les annulations, les données manquantes sur les champs utiles, les quantités invalides ou non positives et les codes de frais, remises et ajustements. Les données nettoyées conservent le pays et la date de vente. Une facture forme un panier, dans lequel chaque code produit distinct apparaît une fois.

Après nettoyage, les 522 415 lignes de vente retenues donnent **19 942 transactions** et **3 831 articles distincts**.

| Facture | Articles distincts | Produits du panier (extrait) |
| --- | ---: | --- |
| 536365 | 7 | WHITE HANGING HEART T-LIGHT HOLDER, WHITE METAL LANTERN, CREAM CUPID HEARTS COAT HANGER |
| 536366 | 2 | HAND WARMER UNION JACK, HAND WARMER RED POLKA DOT |
| 536367 | 12 | ASSORTED COLOUR BIRD ORNAMENT, POPPY'S PLAYHOUSE BEDROOM, POPPY'S PLAYHOUSE KITCHEN |

Le notebook affiche les trois paniers complets. Le CSV fournit les lignes nettoyées et la liste Python `transactions` contient les paniers pour FP-Growth.
