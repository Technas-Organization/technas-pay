# Ce qui est exigé, et quand

Technas Pay ne vous demande rien pour commencer. Les pièces arrivent au moment où l'argent doit sortir, pas avant. Lu dans le code du serveur le 15 novembre 2026.

## Étape par étape

| Étape | Ce qu'on vous demande | Ce que ça débloque |
|---|---|---|
| Tester | Rien. Écrivez-nous, vous recevez une clé `tps_test_` | Tout le parcours, sans argent réel : paiement réussi, refusé, expiré, remboursé, webhooks |
| Encaisser en mobile money | Un compte réel ouvert par Technas et sa clé `tps_live_` | Vos clients paient ; l'argent s'ajoute à votre solde |
| Être reversé en mobile money | Un numéro de reversement par devise, confirmé par e-mail, **et** une pièce d'identité du dirigeant | Les reversements, jusqu'à 100 000 000 F CFA par mois |
| Être reversé sans plafond | RCCM et IFU (ou NIF) de votre société | Les reversements sans plafond mensuel |
| Encaisser par carte | L'inscription de votre compte carte, faite chez Stripe | Les paiements par carte, versés par Stripe sur votre banque |

## Encaisser : aucun papier

Aucun plafond ne bloque un paiement, et aucune pièce n'est exigée pour encaisser. Un plafond atteint ferait échouer vos ventes du jour au lendemain : nous l'avons écarté.

Nous vous demanderons quand même votre RCCM par écrit. C'est une obligation de notre côté : montrer que nous connaissons nos partenaires. Votre réponse, ou son absence, ne bloque aucun paiement.

## Recevoir l'argent du mobile money : votre identité

Le niveau d'identité de votre compte (`kyc_level`) décide seulement de ce qui peut **sortir** :

| Niveau | Ce que nous avons | Reversements |
|---|---|---|
| `none` | Rien | Aucun. L'argent attend sur votre solde, il ne se perd pas |
| `identity` | Pièce d'identité du dirigeant | Jusqu'à 100 000 000 F CFA par mois civil (l'équivalent au cours du jour pour une autre monnaie) |
| `business` | RCCM et IFU | Sans plafond |

La pièce d'identité n'est pas gardée dans notre base de données, mais dans un coffre à secrets à part.

### Le numéro de reversement

Un numéro par devise. Vous le fixez par l'API (`setPayoutDestination`), et **le responsable de votre compte** le confirme par un lien reçu par e-mail. La clé d'API seule ne suffit pas : quelqu'un qui volerait votre clé ne pourrait pas détourner vos reversements.

- Premier numéro d'une devise : actif dès le clic sur le lien.
- Changement de numéro : actif 48 heures après le clic, et vous êtes prévenu deux fois (avis `payout_destination.change_scheduled`, puis `.updated`).

Voir aussi : [Versements](versements.md).

## La carte : votre compte Stripe

Pour la carte, Technas crée votre compte Stripe et vous envoie vers l'inscription hébergée par Stripe (`startCardOnboarding`). C'est Stripe qui vous demande votre identité et votre compte bancaire, selon ses propres règles et celles de votre pays. Technas ne voit pas votre IBAN.

Une fois l'inscription finie, vous avez un compte Stripe complet : vous vous connectez à https://dashboard.stripe.com/ avec vos identifiants, vous réglez vos virements et vous suivez vos litiges. Les frais Stripe sont à votre charge, selon votre contrat Stripe.

État au 15 novembre 2026 : la carte fonctionne en mode test. En réel, elle ouvrira quand le compte Stripe de Technas Pay sera activé.

## Passer en réel

Écrivez à contact@technas.fr. Nous ouvrons votre compte réel et vous remettons votre clé `tps_live_`. Votre code ne change pas : seule la clé change.
