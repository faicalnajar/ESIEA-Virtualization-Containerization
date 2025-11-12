# LAB - Docker images

Dans ce LAB, vous vous familiariserez avec les ***Dockerfile*** : un outil vous permettant de `packager` vos applications dans des images docker réutilisables.

Grâce aux `Dockerfiles`, vous serez en mesure de figer votre application à un instant T, dans une image docker qui vous servira par la suite à créer des containers.

Jusqu'ici, vous avez appris à utiliser les différentes ressources `docker`, pour créer des environnements en adressant les différentes problématiques de :

- Configuration (des containers)
- Stockage (volume)
- Réseau Network

Tout cela, en utilisant des `images docker` proposées par d'autres utilisateurs (`nginx`, `node:alpine`, `madjidtaoualit/uno`). Maintenant, il est temps d'apprendre à créer vos propres images.

L'objectif de ce LAB est de vous apprendre à utiliser le fichier `Dockerfile` pour construire des images docker à partir d'une application, vous permettant ainsi de porter cette application en tout simplicité vers de multiples environnements.

Ainsi, le jour où vous serez amené face à des problématiques de `mise en production` d'une application, vous aurez les cartes en main pour livrer votre produit sous la forme la plus `portable` possible : Une image Docker. Et ainsi potentiellement la déployer sur de l'infrastructure, avec les connaissances que vous avez accumulées jusqu'à présent sur Docker.

