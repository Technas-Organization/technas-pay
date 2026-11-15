# Versements

Comment l'argent encaissé vous revient. Lu dans le code du serveur et vérifié en production le 15 novembre 2026.

## Mobile money : votre solde, puis votre numéro

Un paiement mobile money réussi crédite **votre solde Technas**, dans la devise payée, moins la commission. Vous le lisez avec `getBalance()`.

Chaque lundi à 9 h (UTC), Technas reverse ce solde sur votre numéro de reversement, quand :

- le solde de la devise dépasse 10 000 F CFA (ou l'équivalent au cours du jour) ;
- un numéro de reversement est confirmé dans cette devise ;
- le niveau d'identité du compte le permet, dans la limite de son plafond mensuel (voir [Ce qui est exigé](ce-qui-est-exige.md)).

**Les frais du versement sont à votre charge.** Ce sont ceux de l'opérateur du numéro qui reçoit ; ils sont retirés de la somme envoyée. Votre solde baisse du montant reversé, votre numéro reçoit ce montant moins les frais.

Chaque versement se suit par l'API (`getSettlements()`). La somme quitte votre solde avant l'envoi. Si le versement échoue pour de bon, elle y revient. Un refus passager (côté opérateur ou côté prestataire) laisse le versement en attente ; il est retenté toutes les 10 minutes, sans risque d'envoi en double.

Hors zone franc CFA (mode `local`), le solde reste dans la monnaie payée et part sur votre numéro dans cette monnaie.

## L'état réel aujourd'hui

**Au 15 novembre 2026, aucun reversement mobile money vers un partenaire n'est encore parti.**

Les encaissements fonctionnent. Les reversements sont branchés et testés de notre côté, mais le prestataire mobile money par lequel ils passent n'a pas encore ouvert les versements vers des numéros tiers sur notre compte : ses réponses sont des refus `payout_not_enabled`. Son support nous a confirmé que ces versements sont prévus, et vérifie notre compte.

Ce que ça veut dire pour vous :

- votre argent encaissé reste sur votre solde, visible par l'API ;
- rien ne se perd : il partira au premier lundi qui suit l'ouverture ;
- nous ne donnons pas de date, parce qu'elle ne dépend pas de nous.

Cette section sera mise à jour le jour où le premier reversement part.

## Carte : Stripe vous paie directement

L'argent d'un paiement par carte ne passe pas par votre solde Technas. Le client paie votre compte Stripe, Stripe retient la commission Technas au passage, et Stripe vire le reste sur votre banque, au rythme que vous réglez dans votre tableau de bord Stripe. Les paiements par carte n'apparaissent donc pas dans `getBalance()`.

## Remboursements

- **Mobile money** : `refund` renvoie l'argent au numéro qui a payé, depuis votre solde. Les frais du remboursement sont à votre charge par défaut (`fees: "partner"`), ou retirés de ce que reçoit le client (`fees: "customer"`). La commission n'est pas rendue. Un remboursement passe par le même canal que les versements : tant que celui-ci n'est pas ouvert, il attend lui aussi.
- **Carte** : `refund` rembourse la carte depuis votre compte Stripe. La commission n'est pas rendue.
- Un remboursement fait hors de Technas (espèces, autre portefeuille) s'enregistre avec `recordExternalRefund`, pour que vos comptes restent justes.
