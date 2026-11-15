# Technas Pay

**Le mobile money et la carte, avec un seul SDK.** Vos clients d'Afrique de l'Ouest et du Centre paient par Orange Money, MTN MoMo, Moov Money, Wave ou Mixx, ceux d'ailleurs par carte, et votre code ne fait la différence qu'avec un paramètre : `method: "mobile_money"` ou `method: "card"`.

- **Du serveur jusqu'à l'app Flutter, déjà écrit.** Votre serveur monte des routes toutes faites, votre app Flutter affiche le paiement, l'attente, le refus et l'expiration. Une vingtaine de lignes de chaque côté. Aucune clé dans l'app.
- **Votre assistant de code sait l'intégrer.** Chaque SDK livre un fichier `AGENTS.md` : dites à Codex ou à Claude Code de le lire, il branche le paiement dans votre projet.
- **Carte : 1 % (plus les frais Stripe de votre compte). Mobile money : 2 % (les frais de l'opérateur sont payés par le client).** Sans abonnement, sans frais d'installation, sans minimum. [Détail des frais](#ce-que-ça-coûte).
- **Le mode test, sans papier.** Une clé de test suffit pour faire tout le parcours : réussi, refusé, expiré, remboursé, webhooks reçus. En réel, le code est le même.

## Les SDK

| Votre serveur ou votre app | SDK |
|---|---|
| Next.js (App Router) | [technas-pay-next](https://github.com/Technas-Organization/technas-pay-next) |
| Node.js : Express, Fastify, NestJS, Hono, Bun, Deno, serverless | [technas-pay-node](https://github.com/Technas-Organization/technas-pay-node) |
| Python : FastAPI, Django, Flask | [technas-pay-python](https://github.com/Technas-Organization/technas-pay-python) |
| PHP : Laravel, Symfony, Slim, PHP sans framework | [technas-pay-php](https://github.com/Technas-Organization/technas-pay-php) |
| Dart côté serveur : shelf, Dart Frog | [technas-pay-dart](https://github.com/Technas-Organization/technas-pay-dart) |
| App Flutter (Android, iOS, web) | [technas_pay_flutter](https://github.com/Technas-Organization/technas-pay-dart/tree/main/technas_pay_flutter) |

Les cinq SDK serveur ont la même API, et rejouent le même contrat de routes que le client Flutter. Vous pouvez changer de langage côté serveur sans toucher à l'app.

## Démarrer en 5 minutes

**1. Demandez une clé de test** à contact@technas.fr. Elle commence par `tps_test_` et ne déplace aucun argent.

**2. Installez le SDK** de votre serveur, par exemple en Node :

```bash
npm install github:Technas-Organization/technas-pay-node#v0.11.0
```

```bash
TECHNAS_PAY_SECRET=tps_test_...
```

**3. Demandez un paiement** et envoyez le client sur la page de paiement :

```js
import { createTechnasPay, retryReference } from "@technas/pay";

const pay = createTechnasPay({ secret: process.env.TECHNAS_PAY_SECRET });

app.post("/orders/:id/pay", async (req, res) => {
  const order = await db.orders.find(req.params.id);          // le montant vient de VOTRE base
  const { checkoutUrl } = await pay.askForPayment({
    amount: order.amount,                                     // 5000 = 5 000 F CFA
    reference: retryReference(order.id, order.attempt),       // une référence = un essai
    customer: { email: order.email, name: order.name },
    returnUrl: "https://votre-site.com/merci",
  });
  res.redirect(303, checkoutUrl);
});
```

**4. Recevez le webhook** : la commande est payée quand `onCompleted` arrive, jamais sur la seule foi du retour du navigateur. Chaque SDK montre comment le monter.

**5. Payez en test.** La page de test propose « réussi » ou « refusé ». Par carte, en test : `4242 4242 4242 4242`, une date future, n'importe quel code.

Pour une app Flutter, le serveur monte les routes toutes faites et l'app les appelle :

```dart
final technasPay = TechnasPayClient(
  baseUrl: 'https://api.votre-app.com/api/technas-pay',
  headers: () async => {'Authorization': 'Bearer ${await session.token()}'},
);
final result = await showTechnasPayment(context, backend: technasPay, order: {'orderId': order.id});
```

Le détail est dans le README de chaque SDK.

## Comprendre les paiements

- **[Devises et pays](docs/devises-et-pays.md)** : les deux francs CFA (`XOF` et `XAF`), la devise choisie d'après le pays, les monnaies hors zone, la carte en EUR ou USD.
- **[Ce qui est exigé, et quand](docs/ce-qui-est-exige.md)** : rien pour tester ni pour encaisser ; une pièce d'identité pour être reversé.
- **[Versements](docs/versements.md)** : comment l'argent vous revient, et où en sont les reversements mobile money aujourd'hui.

## Ce que ça coûte

| | Commission Technas | Autres frais |
|---|---|---|
| Mobile money | 2 % du paiement réussi | Les frais de l'opérateur sont payés par le client, ajoutés au prix. Les frais de chaque reversement vers votre numéro sont à votre charge |
| Carte | 1 % du paiement réussi | Les frais Stripe, à votre charge, selon votre contrat Stripe |

Pas d'abonnement, pas de frais d'installation, pas de minimum. La commission n'est pas rendue sur un remboursement. Un paiement refusé ou expiré ne coûte rien.

## Ce qui est en production, et ce qui ne l'est pas

Au 15 novembre 2026 :

| | État |
|---|---|
| Encaissement mobile money, zone franc CFA | En production |
| Mode `local` hors zone franc CFA | En production, à activer vous-même |
| Carte | Fonctionne en mode test. En réel, ouvre quand le compte Stripe de Technas Pay sera activé |
| Reversements mobile money | Branchés, mais pas encore ouverts par notre prestataire mobile money. Votre argent attend sur votre solde. [Détail](docs/versements.md#létat-réel-aujourdhui) |
| Change des monnaies hors zone en francs CFA (mode `converted`) | Pas encore ouvert |
| Paquets npm, PyPI et pub.dev | Pas encore publiés : installation depuis GitHub |

## Comparé aux autres

Des prestataires solides existent déjà en Afrique. Voici ce qu'ils publient eux-mêmes, relevé le 15 novembre 2026, à côté de Technas Pay. Vérifiez sur leurs pages : leurs offres changent.

| | Commission mobile money | Commission carte | Abonnement | SDK Flutter officiel | Pour encaisser en réel | Reversement |
|---|---|---|---|---|---|---|
| **Technas Pay** | 2 % ; frais de l'opérateur payés par le client, en plus | 1 % + frais Stripe | Non | Oui, avec les routes serveur déjà écrites | Aucune pièce (une pièce d'identité pour être reversé) | Chaque lundi dès 10 000 F CFA ; **pas encore ouvert** |
| CinetPay | Grille officielle non lisible sans navigateur ([tarifs](https://cinetpay.com/tarifs)) | idem | Non trouvé | Oui ([GitHub](https://github.com/cinetpay)) | Pièces de la société, établie dans le pays ([FAQ](https://cinetpay.com/faq)) | Après une vérification d'identité complète |
| FedaPay | 1,8 % sur MTN et Moov pour l'offre Tap & Go, Bénin ([FAQ](https://docs.fedapay.com/tap-go/en/faq-en)) ; autres taux non publiés sur la page lue | Non publié sur la page lue | Aucun frais mensuel ([plugin WordPress](https://wordpress.org/plugins/woo-gateway-fedapay)) | Non trouvé ([GitHub](https://github.com/fedapay)) | CNI + IFU + RCCM pour une entreprise ([doc](https://docs.fedapay.com/fr/introduction/compte)) | Non lu sur une page accessible |
| KKiaPay | 1,5 % (la même page affiche aussi 1,3 %) ([tarifs](https://kkiapay.me/pricing)) | 2,1 % UEMOA, 3 % hors UEMOA | 9 900 F CFA par mois, premier mois offert | Oui ([GitHub](https://github.com/kkiapay)) | Au Bénin : RCCM + IFU + pièce d'identité, ou IFU seul pour un consultant ([doc](https://docs.kkiapay.me/v1/compte/activation-de-compte)) | Tous les 3 jours, chaque semaine ou chaque mois ([FAQ](https://kkiapay.me/faq/?lang=en)) |
| PayDunya | 2 % à 2,25 % selon le pays ([frais](https://paydunya.com/service-fees)) | 3,5 % | Non | Non trouvé ([GitHub](https://github.com/paydunyadev)) | Pièce d'identité ; registre de commerce et NINEA pour une entreprise ([FAQ](https://paydunya.com/faq)) | Sous 72 h ouvrables dans la même zone ([FAQ](https://paydunya.com/faq)) |
| Paystack (Côte d'Ivoire) | 1,95 % HT ([article](https://support.paystack.com/en/articles/2130306)) | 3,2 % HT (carte locale), 3,8 % HT (internationale) | Non mentionné | Oui ([GitHub](https://github.com/PaystackOSS)) | Offre « Starter » : pièce d'identité, justificatif de domicile, plafond de 3 M XOF | J+2 ouvrés en XOF |
| Flutterwave (Côte d'Ivoire) | 2,5 % HT ([tarifs](https://flutterwave.com/ci/pricing)) | 3,5 % HT (locale), 4,8 % HT (internationale) | Non mentionné | Oui ([GitHub](https://github.com/Flutterwave)) | Documents de la société et pièce des dirigeants ([doc](https://developer.flutterwave.com/docs/quickstart)) | Le lendemain pour un paiement local |

Les taux ne se comparent pas toujours ligne à ligne : selon le prestataire, les frais de l'opérateur sont compris dans la commission ou payés en plus par le client, et les taux affichés hors taxes ne le sont pas partout. CinetPay et FedaPay ne publient pas leur grille sur une page lisible sans navigateur : demandez-la-leur.

Ce que Technas Pay apporte en plus, c'est surtout le temps d'intégration : les routes serveur et les écrans Flutter sont déjà écrits, et un assistant de code peut tout brancher en lisant `AGENTS.md`. Ce qu'il n'a pas encore : l'ancienneté, des reversements mobile money ouverts et la carte en réel.

## Contact

contact@technas.fr. Présentation : [pay.technas.fr](https://pay.technas.fr). Technas Pay est un service de [Technas](https://technas.fr).
