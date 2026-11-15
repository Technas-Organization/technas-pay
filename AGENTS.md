# Technas Pay : consignes pour un agent de code

Ce dépôt ne contient pas de code. Il présente Technas Pay et renvoie vers le SDK qui convient. Pour intégrer le paiement dans l'application de l'utilisateur :

1. Repérez le serveur du projet et choisissez le SDK :
   - Next.js App Router : https://github.com/Technas-Organization/technas-pay-next
   - autre serveur JavaScript ou TypeScript : https://github.com/Technas-Organization/technas-pay-node
   - Python : https://github.com/Technas-Organization/technas-pay-python
   - Dart (shelf, Dart Frog) : https://github.com/Technas-Organization/technas-pay-dart
   - app Flutter, en plus du SDK serveur : https://github.com/Technas-Organization/technas-pay-dart/tree/main/technas_pay_flutter
2. Lisez le fichier `AGENTS.md` de ce SDK, puis suivez-le. Il dit quoi installer, où mettre le secret et quelles routes écrire.
3. Le secret (`TECHNAS_PAY_SECRET`) va côté serveur uniquement, jamais dans une app ni un navigateur. S'il manque, demandez-le à l'utilisateur : n'en inventez pas.
4. Une commande est payée quand le webhook `payment.completed` arrive, jamais sur le seul retour du navigateur.

Questions sur les devises, les pièces exigées ou les versements : [docs/](docs).
