# Indications pour le LAB

Pour accomplir les deux exercices décrits dans ce LAB, vous devez créer un Dockerfile pour chaque application et suivre les étapes décrites. Voici comment vous pouvez les accomplir :

## Exercice: Partie 1

- Téléchargez l'application Node.js depuis le lien fourni dans l'exercice. Vous devriez avoir un dossier contenant les fichiers de l'application.

- Créez un fichier `Dockerfile` dans le même répertoire que les fichiers de l'application.

- Éditez le `Dockerfile` pour qu'il ressemble à ceci :

```Dockerfile
# Étape 1: Utilisez l'image node:alpine comme base
FROM node:alpine

# Étape 2: Définissez le répertoire de travail
WORKDIR /home/node

# Étape 3: Copiez le fichier package.json et le fichier package-lock.json pour installer les dépendances de production
COPY package*.json ./

# Étape 4: Installez les dépendances de production
RUN npm install --only=production

# Étape 5: Copiez l'ensemble du code de l'application dans le répertoire de travail
COPY . .

# Étape 6: Définissez la commande par défaut pour démarrer l'application
CMD ["npm", "run", "start"]
```

- Après avoir créé le Dockerfile, assurez-vous d'être dans le même répertoire que le Dockerfile, puis exécutez la commande suivante pour construire l'image Docker :

```bash
docker build -t lab-images:latest .
```
- Une fois l'image `Docker` construite avec succès, vous pouvez la tester en exécutant le conteneur Docker :

```bash
docker run -p 3000:3000 lab-images:latest
```
- Accédez à l'application `Node.js` depuis un navigateur en ouvrant http://localhost:3000. Vous devriez voir l'interface "express website".

- Pour vérifier la sensibilité à la variable d'environnement PORT, exécutez la commande suivante avec un port différent :

```bash
docker run --rm -e PORT=2000 -p 3000:2000 lab-images:latest
```
- Ouvrez votre navigateur sur http://localhost:3000 à nouveau, et vous devriez toujours voir le même résultat.




