# LAB - DOCKER COMPOSE PRODUCTION

Ce LAB servira à valider vos connaissances sur l'ensemble de `Docker` (vu jusqu'ici) à savoir :

- Run des containers et les configurer
- volume
- network
- images (Dockerfile)
- Docker-compose

Les différents composants de l'exercice :

- Container Base de donnée `MySQL`
- Container PhpMyadmin pour administrer la base `MySQL`
- Container node contenant une `API express` (la même que pour le tp [LAB_05_DOCKER_COMPOSE_DEV](../LAB_05_DOCKER_COMPOSE_DEV), qui va se connecter à la base `MySQL` grâce à des variables d'environnements, pour afficher une liste des messages enregistrés en base.
- Container `nginx` contenant le `front (React)`, connecté à l'`API back-end` pour afficher la liste de messages (récupérés en base, par le back-end).

Dans cet exercice, vous allez être amené à :

- Développer des fichiers `Dockerfile` pour l'`API backend (node)`, et le `front-end (React)` qu'il va falloir servir derrière un serveur `nginx`.
- Configurer l'environnement à l'aide de `docker-compose`.


Utiliser la commande `build` dans votre fichier `docker-compose.yml` pour préciser un chemin vers un `Dockerfile` à construire :

```yml

version: "3"
services:
  myservice:
    # Lorsqu'on utilise build, l'image ici sera le nom de l'image créer lors du build (comme pour docker build -t <mon-image>)
    image: monimage
    # build le Dockerfile présent dans ./app
    build: ./app
    # ...
    
```

**Pour rappel: les applications front-end dépendantes d'une phase de `build` (Webpack, gulp...) comme celle utilisée dans cet exercice nécessite deux `stages` pour le build**:
- `node`: Installation des dépendances `NPM`, `npm run build` (ou équivalent) pour construire les fichiers statiques du site
- `nginx` (ou autre serveur web) pour servir le contenu statique (buildés dans l'étape précédente).

**Attention, la partie `client` (front) utilise une variable d'environnement pour spécifier l'hôte du backend, il s'agit de `REACT_APP_BACKEND_URL`, doit être spécifiée lorsqu'on va exécuter le container**

Pour cet exercice, je vous invite à ré-utiliser le fichier `docker-compose.yml` produit lors du précédent [LAB_05_DOCKER_COMPOSE_DEV](../LAB_05_DOCKER_COMPOSE_DEV), puisque la configuration de MySQL et phpmyadmin reste la même (ou va changer très peu).

Vous avez vu au cours du [LAB_04_DOCKER_IMAGES](../LAB_04_DOCKER_IMAGES) comment faire builder :
- 1 image pour une ```API``` back-end ```node.js```
- 1 image nécessitant plusieurs stages de build (par exemple : ```uno```).

## Les consignes

### Veuillez dockeriser [cette application](https://github.com/nailtonvital/react-node-mysql-crud).

### MySQL

- Le serveur `MySQL` ne doit pas être accessible depuis l'hôte (seulement à travers l'`API back-end`, et `phpmyadmin`)
- Les données générées par le container `MySQL` doivent être persistées en local.

### Phpmyadmin

- `Phpmyadmin` doit être disponible sur le port `8080` de la machine
- `Phpmyadmin` doit être connecté/authentifié au container `MySQL`.

### API Back-End

- L'`API Back-end` ne doit pas être accessible depuis la machine hôte
- L'`API Back-end` doit être connectée au container `MySQL`, **Mais pas au container phpmyadmin**.
- L'`API Back-End` doit être configuré pour requêter la Base de données via les variables d'environnements suivant:

  - `DB_HOST` : Hôte (nom du container) de `MySQL`, par défaut sur `127.0.0.1`
  - `DB_USER` : Utilisateur de la base de donnée `MySQL`, par défaut `test`
  - `DB_PASSWORD` : Mot de passe de le base de donnée `MySQL`, par défaut `test`
  - `DB_NAME` : Le nom de la base à utiliser sur le serveur `MySQL`, par défaut `mydatabase`

### Front end

- Le `Front-End` doit être disponible sur le port `80` de l'hôte
- Le `Front-end` doit être connecté à l'`API backend` (mais pas à `MySQL` et `Phpmyadmin`)
- Le `Front-end` doit être configuré pour requêter l'`API back-end` (à travers la variable d'environnement `REACT_APP_BACKEND_URL`).

## Vérification

Pour vous assurez que l'exercice est réalisé correctement, vous devrez ouvrir votre navigateur sur le `localhost:80`.


## Elements de réponses 

Pour résoudre les problèmes rencontrés dans votre projet (erreurs de CORS, absence de données dans la base, etc.), voici les points importants à corriger et à comprendre.

**1. Configurer un Proxy avec Nginx**

- Problème : Les requêtes de l'application frontend vers le backend sont bloquées par la politique de Same-Origin Policy car le frontend fait des requêtes directement au backend (http://server:3001). Cela cause une erreur CORS.

- Solution : Configurer un proxy dans Nginx pour rediriger les requêtes API (/api) du frontend vers le backend.

- Pourquoi : Cela évite les problèmes de CORS en faisant en sorte que toutes les requêtes (API et frontend) soient servies depuis la même origine (http://localhost).

**Fichier nginx.conf**

Voici un exemple de configuration Nginx pour servir les fichiers statiques du frontend et proxy les requêtes vers le backend :

```
server {
    listen 80;

    # Serveur les fichiers statiques du frontend
    location / {
        root /usr/share/nginx/html;
        index index.html;
        try_files $uri /index.html;
    }

    # Proxy des requêtes API vers le backend
    location /api/ {
        proxy_pass http://server:3001/; # Rediriger vers le service backend
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

**2. Dockerfile pour le Frontend**

Voici un exemple de Dockerfile pour le frontend qui utilise Nginx pour servir l'application :

```
# Utiliser l'image Nginx
FROM nginx:alpine

# Copier les fichiers construits par Vite (dist) dans le dossier Nginx
COPY ./dist /usr/share/nginx/html

# Ajouter la configuration personnalisée pour Nginx
COPY ./nginx.conf /etc/nginx/conf.d/default.conf

# Exposer le port 80 pour servir l'application
EXPOSE 80

# Lancer Nginx
CMD ["nginx", "-g", "daemon off;"]
```


**3. Créer la Base de Données Avant de Tester**

- Problème : Si la table ou la base de données n’est pas créée, les requêtes au backend (http://server:3001/games) ne renverront pas de données.

- Solution : Créer la base de données et la table avant de tester.

Étapes :

1. Accéder au Conteneur MySQL :

```
docker exec -it <nom-du-conteneur-database> bash
```

2. Se connecter à MySQL :

```
mysql -u root -p
```

3. Saisissez le mot de passe configuré dans le fichier docker-compose.yml.

Créer la Table :

```

CREATE DATABASE IF NOT EXISTS crudgames;
USE crudgames;

CREATE TABLE IF NOT EXISTS games (
    id INT AUTO_INCREMENT PRIMARY KEY,
    name VARCHAR(255) NOT NULL,
    genre VARCHAR(255) NOT NULL,
    release_date DATE
);
```
Ajouter des Données pour Tester :

```
INSERT INTO games (name, genre, release_date) VALUES ('Game 1', 'Action', '2024-01-01');
```

**4. Tester l’Application**

Depuis le Conteneur Frontend

Une fois la base de données et le backend correctement configurés, vous pouvez tester les requêtes directement depuis le conteneur frontend :

1. Accéder au Conteneur Frontend :
```
docker exec -it <nom-du-conteneur-frontend> sh
```

2. Effectuer une Requête via curl :
```
curl -v http://server:3001/games
```
Cela devrait renvoyer les données disponibles dans la table games.

**5. Résumé des Étapes pour Tester**

- Configurer Nginx pour proxy les requêtes API.
- Construire l’application frontend et utiliser Nginx pour la servir.
- Créer la base de données et ajouter des données à la main.
- Tester que le backend renvoie les données à travers l’URL http://server:3001/games
