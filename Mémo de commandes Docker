# Mémo de commandes Docker

- ``docker run alexwhen/docker-2048`` - Permet de lancer une machine Docker.
(Remplacer "alexwhen/docker-2048" par un container trouvé sur le [Hub Docker](https://hub.docker.com)).  

- ``docker run -d alexwhen/docker-2048`` (``-d``) - Permet de détacher le container et donc d'en lancer plusieurs simultanément.  

- ``docker run -p 8080:80 alexwhen/docker-2048`` (``-p 8080:80``) - Permet d'ouvrir un port réseau sur la machine hôte vers votre container Docker. (Vous pouvez remplacer "8080:80" par un autre port). 

- ``docker pull alexwhen/docker-2048`` - Permet récupérer des images sur le Docker Hub sans pour autant lancer de conteneur.  

- ``docker ps`` - Permet de voir les containers en cours de fonctionnement ainsi que leurs détails.  

- ``docker ps -a`` - Permet de voir l'ensemble des containers ainsi que leurs détails présents sur votre ordinateur.  

- ``docker exec -ti 8ef14db41758`` - Permet d'ouvrir un shell dans le container pour executer des commandes.
(Remplacer "8ef14db41758" par l'ID du container désiré). 

- ``docker stop 8ef14db41758`` - Permet d'arrêter un container en cours de fonctionnement.
(Remplacer "8ef14db41758" par l'ID du container désiré).  

- ``docker system prune`` - Permet de supprimer les containers arrêtés et les images non utilisées.  

- ``docker build -t ocr-docker-build .`` - Permet de construire une image docker.  
L'argument "``-t``" permet de donner un nom à l'image docker.  
Le "``.``" défini le répertoire où se trouve le Dockerfile.

---

# Mémo sur les Dockerfiles

> Un dockerfile est un fichier contenant une suite d'instruction et permettant de créer une image Docker.

- ``FROM debian:9`` (``FROM``) - Permet de définir l'image docker utilisé comme base.
(Remplacer "debian:9" par l'image que vous désiré).  

- ``RUN apt-get update -yq`` (``RUN``) - Permet d'exécuter des commandes dans votre container.
Exemple :
````dockerfile
    RUN apt-get update -yq \
    && apt-get install curl gnupg -yq \
    && curl -sL https://deb.nodesource.com/setup_10.x | bash \
    && apt-get install nodejs -yq \
    && apt-get clean -y
````  

- ``ADD . /app/`` (``ADD ``) - Permet d'ajouter des fichiers / dossiers dans votre container.
En l'occurence, nous recupérons les différents éléments du dossier courant "``.``" et les envoyons dans le dossier "``/app/``" du container.

- ``WORKDIR /app`` (``WORKDIR``) - Permet de modifier le répertoire courant (équivalent à la commande "``cd``").
L'ensemble des commandes qui suivront seront toutes exécutées depuis le répertoire défini.  

- ``EXPOSE 2368`` (``EXPOSE``) - Permet d'indiquer le port sur lequel votre application écoute. (Remplacer "2368" par le port désiré).

- ``VOLUME /app/logs`` (``VOLUME``) - Permet d'indiquer quel répertoire vous voulez partager avec votre host. (Remplacer "/app/logs" par le port désiré).  

- ``CMD npm run start`` (``CMD``) - Permet au container de savoir quelle commande il doit exécuter lors de son démarrage.  

Exemple du Dockerfile final :
```dockerfile
FROM debian:9

RUN apt-get update -yq \
&& apt-get install curl gnupg -yq \
&& curl -sL https://deb.nodesource.com/setup_10.x | bash \
&& apt-get install nodejs -yq \
&& apt-get clean -y

ADD . /app/
WORKDIR /app
RUN npm install

EXPOSE 2368
VOLUME /app/logs

CMD npm run start
```
