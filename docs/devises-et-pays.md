# Devises et pays

Ce qui suit est lu dans le code du serveur Technas Pay, le 15 novembre 2026. Quand la liste change, cette page change avec elle.

## En bref

| Vous faites payer… | Devise | Décimales | Ce qu'il faut |
|---|---|---|---|
| En mobile money, zone franc CFA | `XOF` ou `XAF`, choisie d'après le pays | Aucune | Rien : c'est le réglage de départ |
| En mobile money, hors zone franc CFA | La monnaie du pays (cedi, naira…) | 0 ou 2 selon la monnaie | Activer le mode `local` et accepter le risque de change |
| Par carte | Toute devise acceptée par votre compte carte (EUR, USD, XOF…) | Celles de la devise | Un compte carte ouvert |

## Deux francs CFA, pas un

Le franc CFA existe sous deux codes. Ce sont deux monnaies distinctes, émises par deux banques centrales, mais à la **même parité fixe avec l'euro** (655,957 F pour 1 €). Un montant de 5 000 vaut donc la même chose dans les deux, sans conversion.

| Zone | Code | Pays |
|---|---|---|
| UEMOA (Afrique de l'Ouest) | `XOF` | Bénin `BJ`, Burkina Faso `BF`, Côte d'Ivoire `CI`, Guinée-Bissau `GW`, Mali `ML`, Niger `NE`, Sénégal `SN`, Togo `TG` |
| CEMAC (Afrique centrale) | `XAF` | Cameroun `CM`, Centrafrique `CF`, Congo `CG`, Gabon `GA`, Guinée équatoriale `GQ`, Tchad `TD` |

Cette table est la même partout : `ZONE_CFA` dans le serveur, `CURRENCY_BY_COUNTRY` dans les SDK (`currencyForCountry("CM")` rend `"XAF"`).

### La devise se choisit toute seule

Vous donnez le montant. La devise est prise dans cet ordre :

1. `currency`, si vous la passez ;
2. sinon celle du pays du client (`customer.country`, code ISO à deux lettres) ;
3. sinon `defaultCurrency`, réglé à la création du client SDK ;
4. sinon le client choisit son pays sur la page de paiement. Son numéro de téléphone, s'il est au format international, préremplit ce choix.

Votre solde et le webhook suivent **la devise réellement payée**. Un client camerounais crédite votre solde en `XAF`, un client togolais en `XOF`. Chaque devise a son solde et son numéro de reversement.

### Les pays ouverts aujourd'hui

Un pays n'est proposé que si au moins un opérateur y encaisse. Relevé le 15 novembre 2026 :

| Pays | Devise | Opérateurs |
|---|---|---|
| Bénin | `XOF` | Celtiis Cash, MTN MoMo, Moov Money |
| Burkina Faso | `XOF` | Moov, Orange |
| Côte d'Ivoire | `XOF` | Djamo, MTN MoMo, Moov Money, Orange Money, Wave |
| Mali | `XOF` | Mobi Cash, Moov Money, Orange Money |
| Niger | `XOF` | Airtel |
| Sénégal | `XOF` | Djamo, Expresso, Free Money, Orange Money, Wave, Wizall, compte PayDunya |
| Togo | `XOF` | Moov Money, Togocel Money |
| Cameroun | `XAF` | MTN MoMo, Orange Money |
| Congo | `XAF` | Airtel, MTN |
| Gabon | `XAF` | Airtel, Moov Money |

Guinée-Bissau, Centrafrique, Guinée équatoriale et Tchad n'ont pas encore d'opérateur. La page les affiche « bientôt » et l'API les refuse (`country_not_available`). La liste des opérateurs est relue toutes les 10 minutes : un pays s'ouvre sans mise à jour de votre côté.

### Pas de décimales en mobile money

Le franc CFA n'a pas de centimes. Un montant mobile money en `XOF` ou `XAF` est un **entier** : `5000` veut dire 5 000 F CFA.

## Hors zone franc CFA : le mode `local`

Vous facturez toujours en francs CFA. Le client paie l'équivalent dans sa monnaie, au cours du jour, et votre solde est crédité dans **cette** monnaie. C'est une option que vous activez vous-même, en acceptant le risque de change : Technas ne le couvre pas et ne prend aucune marge sur le cours.

Pays prévus (`HORS_CFA` dans le serveur) :

| Pays | Devise | Décimales du montant payé |
|---|---|---|
| Ghana `GH` | `GHS` | 2 |
| Nigeria `NG` | `NGN` | 0 |
| Kenya `KE` | `KES` | 0 |
| Guinée `GN` | `GNF` | 0 |
| RD Congo `CD` | `CDF` | 0 |
| Rwanda `RW` | `RWF` | 0 |
| Ouganda `UG` | `UGX` | 0 |
| Tanzanie `TZ` | `TZS` | 0 |
| Zambie `ZM` | `ZMW` | 2 |
| Malawi `MW` | `MWK` | 0 |
| Mozambique `MZ` | `MZN` | 2 |
| Sierra Leone `SL` | `SLE` | 2 |

Un de ces pays n'est proposé au client que si les trois conditions sont réunies :

- un opérateur y encaisse ;
- le cours a moins de 24 heures (72 heures s'il a été publié un vendredi ou le week-end), et deux sources de cours concordent à 2 % près ;
- vous avez un **numéro de reversement confirmé** dans cette monnaie. Sans lui, votre argent n'aurait nulle part où aller.

Le client voit avant de payer le montant dans sa monnaie, le cours et sa source. Ce montant est garanti 30 minutes et arrondi au-dessus. Le webhook donne les deux montants : `amount`/`currency` (ce qui a été payé) et `originalAmount` (votre montant en francs CFA).

Ce qui n'existe pas encore : le mode `converted`, où Technas changerait ces monnaies en francs CFA pour vous. Il est refusé (`fx_mode_unavailable`) tant qu'aucune filière de change n'est en place.

### Le dollar en mobile money : pas aujourd'hui

Aucun paiement mobile money en dollar américain n'est possible. En RD Congo, le mobile money passe en franc congolais (`CDF`). Si vous facturez en dollars, la carte le permet (voir ci-dessous).

## Par carte

La carte suit un autre chemin. Vous avez votre propre compte carte, le client le paie directement, et la commission Technas est retenue au passage. Technas ne détient pas cet argent.

- La devise est **obligatoire** (`currency: "EUR"`, `"USD"`, `"XOF"`…).
- Toute devise acceptée par votre compte carte est possible.
- Les décimales sont celles de la devise : `49.9` pour 49,90 €. Les devises sans centimes (comme `XOF`) restent entières ; quelques devises en ont trois.
- Un montant avec trop de décimales est refusé (`invalid_amount`).

État au 15 novembre 2026 : la carte fonctionne en **mode test**. En réel, elle ouvrira à l'activation du compte carte de Technas Pay. Aucune date n'est promise.

## Où regarder dans votre SDK

- Node : [technas-pay-node, section « La devise »](https://github.com/Technas-Organization/technas-pay-node#la-devise)
- Python : [technas-pay-python](https://github.com/Technas-Organization/technas-pay-python)
- Dart et Flutter : [technas-pay-dart](https://github.com/Technas-Organization/technas-pay-dart)
- Next.js : [technas-pay-next](https://github.com/Technas-Organization/technas-pay-next)
