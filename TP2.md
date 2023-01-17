# Docker  - TP2 : Construction d'images et Registres
> **Objectifs du TP** :
>- Utiliser un registre afin de stocker des images Docker
>- Comprendre comment est construite une image Docker avec son application
>
> **Niveau de difficulté** : Débutant

## 1- Les Images Docker et le Registre distant (Docker Hub)

Nous avons vu dans la première partie que pour lancer un conteneur, nous avons **besoin** d'une image. Il s’agit d’un modèle qui nous permet de créer notre conteneur, en d'autres termes tout ce qu'il contiendra en termes de fichiers et de programmes lors de son lancement. Dans nos exemples, nous avons créé des conteneurs utilisant une image de base Ubuntu.

Comme nous l'avons vu dans le cours, une image est composée de plusieurs couches en lecture seule. Cela permet de créer une hiérarchie réutilisable par tous les conteneurs. Si vous lancez deux conteneurs avec pour image de base Ubuntu, vous n'aurez pas deux fois 400 Mo de fichiers utilisés (comme pour des machines virtuelles), mais bien une fois 400 Mo.

Docker utilise overlay2 ou d'autres systèmes de fichiers comme overlay, BTRFS (comme on peut le voir avec la commande `docker info`) comme système de fichiers. C'est un système de fichiers unifié (_union mount_), qui permet de fusionner plusieurs systèmes de fichiers distincts et de les faire apparaître comme un unique système de fichiers avec une hiérarchie unifiée.

Ainsi plusieurs conteneurs peuvent monter leur système de fichiers avec des briques leur appartenant et d'autres partagées entre tous les conteneurs. Si un de ces conteneurs écrit dans un fichier, il va créer une copie qui lui est propre afin de pouvoir l’utiliser localement après cette modification (mécanisme de _copy on write_).

Les images sont des éléments réutilisables et il en existe pour beaucoup de variétés d'environnements (Java, Python, MySQL, Oracle DB, Postgres, Node.js, etc.). Afin de partager ces environnements, on utilise un **registre** (registry dans le vocabulaire Docker) dans lequel on va stocker ces modèles (et toute la structure de fichiers associée). Imaginez ce registre distant comme un ensemble de dépôts Subversion ou Git. De la même manière qu'on stocke du code, on stocke ici des briques de base constituant nos environnements afin de les réutiliser

## 2- Lister les images docker

Commençons par lister les images disponibles pour notre démon Docker. Les images listées sont celles qui sont directement disponibles localement afin de construire et lancer un conteneur.
```sh
dev $ docker image ls
REPOSITORY  TAG     IMAGE ID      CREATED      SIZE
ubuntu      latest  f753707788c5  4 weeks ago  127.2 MB
```
Les images qui ne sont pas listées doivent être téléchargées à partir du Docker Hub, qui est le registre hébergeant toutes les images de base mises à disposition par Docker et la communauté. N'importe qui peut créer un compte sur le Docker Hub et y stocker des images.

Les images sont stockées dans des dépôts (à la manière d'un repository SVN ou Git) qui sont eux-mêmes stockés dans un registre.
## 3- Télécharger une image Docker

Les images listées par la commande `docker images` ne font pas état d'une image alpine sur notre système. Tentons donc de la télécharger à partir du Docker Hub. Pour ça, on utilise la commande `docker image pull`. Vous devriez observer la sortie suivante :
```sh
dev $ docker image pull alpine
2.6: Pulling from library/alpine
2a3ebcb7fbcc: Pull complete
Digest: sha256:e9cec9aec697d8b9d450edd32860ecd363f2f3174c8338beb5f809422d182c63
2.7: Pulling from library/alpine
4dea34575ff3: Pull complete
Digest: sha256:9f08005dff552038f0ad2f46b8e65ff3d25641747d3912e3ea8da6785046561a
3.1: Pulling from library/alpine
35a9f57fd9f2: Pull complete
Digest: sha256:2d74cbc2fbe3d261fdcca45d493ce1e3f3efd270114a62e383a8e45caeb48788
3.2: Pulling from library/alpine
e052f352ed4b: Pull complete
Digest: sha256:8565a58be8238ef688dbd90e43ec8e080114f1e1db846399116543eb8ef7d7b7
3.3: Pulling from library/alpine
c19324d1d971: Pull complete
Digest: sha256:06fa785d55c35050241c60274e24ad57025683d5e939b3a31cc94193ca24740b
3.4: Pulling from library/alpine
49388a8c9c86: Pull complete
Digest: sha256:915b0ffca1d76ac57d83f28d568bcb516b6c274843ea8df7fac4b247440f796b
3.5: Pulling from library/alpine
b1f00a6a160c: Pull complete
Digest: sha256:b007a354427e1880de9cdba533e8e57382b7f2853a68a478a17d447b302c219c
3.6: Pulling from library/alpine
b56ae66c2937: Pull complete
Digest: sha256:d6bfc3baf615dc9618209a8d607ba2a8103d9c8a405b3bd8741d88b4bef36478
edge: Pulling from library/alpine
e00d546a75ad: Pull complete
Digest: sha256:23e7d843e63a3eee29b6b8cfcd10e23dd1ef28f47251a985606a31040bf8e096
latest: Pulling from library/alpine
Digest: sha256:d6bfc3baf615dc9618209a8d607ba2a8103d9c8a405b3bd8741d88b4bef36478
Status: Downloaded newer image for alpine
```
Docker télécharge plusieurs couches pour l'image de base `alpine`.

Maintenant, découvrons les images téléchargées en relançant la commande `docker image ls`.
```sh
dev $ docker image ls
REPOSITORY          TAG                 IMAGE ID            CREATED             SIZE
alpine              3.5                 6c6084ed97e5        5 months ago        3.99MB
alpine              3.4                 c7fc7faf8c28        5 months ago        4.82MB
alpine              3.3                 fc8815064a1b        5 months ago        4.81MB
alpine              3.2                 220b1d97bf6a        5 months ago        5.27MB
alpine              3.1                 2ba97bb89407        5 months ago        5.05MB
alpine              edge                5c4fa780951b        5 months ago        4.15MB
alpine              3.7                 3fd9065eaf02        5 months ago        4.15MB
alpine              latest              3fd9065eaf02        5 months ago        4.15MB
alpine              3.6                 77144d8c6bdc        5 months ago        3.97MB
alpine              2.7                 93f518ec2c41        2 years ago         4.71MB
alpine              2.6                 e738dfbe7a10        2 years ago         4.5MB
...
```
La commande liste non pas une mais dix versions (images) de la distribution `alpine` à partir de son dépôt stocké sur le [Docker Hub](https://hub.docker.com/_/alpine?tab=tags). On remarque que certaines versions correspondent bien aux couches que l'on a téléchargé précédemment grâce à la commande `docker image pull`. On peut donc choisir une image parmi toutes ces versions afin de démarrer un conteneur.

Lançons un conteneur avec une image de base `alpine`.
```sh
dev $ docker container run -i -t alpine /bin/sh
```
On obtient ainsi un conteneur avec un système de fichiers et des programmes correspondant à la dernière distribution alpine. Depuis l’intérieur du conteneur, vérifions quelle version de alpine a été démarrée avec la commande `cat /etc/os-release`.
```
# cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.8.0
PRETTY_NAME="Alpine Linux v3.8"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
```
Sortez maintenant de ce conteneur en tapant la commande `exit`.

Si on jette de nouveau un coup d'œil sur la sortie de la `commande docker image ls`, on remarque que deux images possèdent le même identifiant ! Comment cela peut-il être possible ? Une image n'est-elle pas censée posséder un identifiant unique ?
```sh
dev $ docker image ls
REPOSITORY                                               TAG       IMAGE ID
[...]
alpine                                                   3.5       a2b04ae28915
alpine                                                   3.4       993b1b41569d
alpine                                                   3.3       19bf2ec565c3
alpine                                                   3.2       e45221eb7f87
alpine                                                   3.1       6f8a01c2945d
alpine                                                   edge      9d1f27787d39
alpine                                                   3.8       11cd0b38bc3c
alpine                                                   latest    11cd0b38bc3c
alpine                                                   3.7       791c3e2ebfcb
alpine                                                   3.6       da579b235e92
[...]
```
En réalité, Docker utilise également des tags pour qualifier les images. Dans notre cas, les images avec les tags `3.8` et `latest` sont en réalité la **même image**. Par défaut, si on met juste `alpine`, Docker construit un nouveau conteneur avec la version marquée par un tag `latest`.

Pour choisir une autre distribution de alpine, par exemple `3.5`, il faut également spécifier le tag de l'image pour la commande docker container run.
```sh
dev $ docker container run -i -t alpine:3.5 /bin/sh
```
Encore une fois, depuis l’intérieur du conteneur, vérifions quelle version de alpine a été démarrée avec la commande `cat /etc/os-release`.
```
# cat /etc/os-release
NAME="Alpine Linux"
ID=alpine
VERSION_ID=3.5.2
PRETTY_NAME="Alpine Linux v3.5"
HOME_URL="http://alpinelinux.org"
BUG_REPORT_URL="http://bugs.alpinelinux.org"
```
Et voilà ! Nous avons cette fois-ci crée un conteneur avec comme image de base une distribution alpine 3.5. Sortez maintenant de ce conteneur en tapant la commande exit.
## 4- Chercher des images

Pour chercher des images disponibles dans le Docker Hub, on peut utiliser la commande `docker search`. Tentons de chercher le terme “mysql” :
```sh
dev $ docker search mysql
```
Vous devriez observer en sortie une grosse quantité de dépôts dans le Docker Hub. Pour chacun d'eux, on peut observer de nombreuses informations comme le nom, la description, les "stars" et les indicateurs "OFFICIAL" et "AUTOMATED".


Le nom identifie l'image. Comme on peut le remarquer, beaucoup de noms sont sous la forme `<namespace>/<nom_repository>` et représentent les dépôts qui sont fournis par la communauté d'utilisateurs de Docker. D'autres sont seulement sous la forme `<nom_repository>` et correspondent aux repositories contenant des images officielles et supportées par l'équipe Docker.

Les Stars montrent la popularité d'une image. Plus elle en a et plus elle est populaire au sein de la communauté. L'indicateur “OFFICIAL” nous indique si l'image est une image officielle et “AUTOMATED” nous informe si l'image a été construite à partir d’un outil de build automatique du Docker Hub (mais nous y reviendrons plus tard. Ce n’est pas la peine de retenir ces informations pour le moment).


A titre d'exemple, téléchargeons une image officielle.
```sh
dev $ docker image pull mysql
```
Puis:
```sh
dev $ docker container run -i -t --name=mysql mysql /bin/bash
```
Puis à l'intérieur du conteneur :
```
# mysql --version
```
La commande nous renvoie bien la version de MySQL installée. Nous n'avons pas besoin de lancer apt-get ou un autre gestionnaire de paquet, MySQL est déjà présent dans notre image.

Sortez maintenant du conteneur en lançant la commande `exit`.

## 5- Supprimer une image

Pour supprimer une image localement sur notre machine, on utilise la commande `docker image rm`.

Par défaut, Docker ne veut pas supprimer une image qui est liée à un conteneur existant que celui-ci soit arrêté ou non. Il faut donc d’abord supprimer le conteneur mysql.
```sh
dev $ docker container rm mysql
```
On peut ensuite supprimer l’image :
```sh
dev $ docker image rm mysql
```
Observez la sortie : on remarque que chacune des couches du système de fichiers sont supprimées les unes après les autres.
## 6- Construction de notre propre image

Maintenant que nous avons vu comment récupérer des images toutes prêtes avec des programmes préinstallés, créons nous même notre propre image personnalisée !
Il existe deux façons de créer notre propre image :

- Grâce à la commande `docker container commit` qui va créer une nouvelle couche de système de fichiers avec les nouveaux fichiers d'un conteneur (par exemple on installe MySQL au dessus d'une image de base ubuntu).

- Avec la commande `docker image build`. Cette commande est différente car une image est construite via une description dans un fichier appelé Dockerfile qui contient une suite d'instruction à exécuter pour construire une image. Par exemple, on peut décrire dans le fichier que l'on doit installer MySQL à partir d'une image de base Ubuntu mais ce n'est pas nous qui interagissons directement avec un conteneur et sauvegardons les changements. C'est Docker lui même qui se charge d'installer MySQL pour nous et qui sauvegarde une nouvelle image !
## 7- Utilisation d’un registre pour faire un commit des changements d'un conteneur

Nous allons maintenant faire un **commit** des changements d'un conteneur que l'on va lancer grâce à l'instruction `docker container commit`. Lançons un conteneur Ubuntu classique.
```sh
dev $ docker container run --name=vimubuntu -i -t ubuntu /bin/bash
```
Vous devriez obtenir, comme d’habitude, une invite de commande de la forme suivante :
```
root@adf131ef9a23:/#
```
Votre conteneur peut être identifié soit par le nom que vous lui avez donné au travers du flag `--name` ou grâce au nombre hexadecimal après root@, ici adf131ef9a23 : c’est l’identifiant court du conteneur.
Effectuons maintenant quelques changements dans notre conteneur.
```
# apt-get update
# apt-get -y install vim
```
Nous venons d'installer vim sur notre conteneur Ubuntu. Cependant, nous savons que si nous tuons le conteneur et que nous le supprimons, nous n'aurons aucun moyen de repartir de ce point sans avoir à retaper ces commandes et télécharger de nouveau le paquet Vim. Pour éviter cela, nous allons faire un commit du conteneur et pousser son image dans un dépôt du registre public docker hub, veuillez d' abord creer un compte sur docker hub http://hub.docker.com/sso/start

Une fois que vous vous êtes enregistré avec succès avec un nom d'utilisateur, toutes les images Docker que vous uploadez sur ce compte sont accessibles par le nom d'utilisateur <VotreNomDeCompte> avec le nom de l'image.

Disons, par exemple, que le nom d'utilisateur de mon compte est dalel. Si une image Docker nommée vimubuntu peut être poussée vers ce compte, alors nous pouvons accéder à l'image comme ci-dessous.
```
docker pull dalel/vimubuntu
```

Tout d'abord, connectez-vous au hub docker en utilisant la commande suivante.

```
docker login
```
Veuillez entrer votre nom d'utilisateur et votre mot de passe ici. Si la connexion est réussie, vous obtiendrez un message comme Login Succeeded.

 ! Tapez exit puis ensuite la commande suivante :
```sh
dev $ docker container commit vimubuntu <VotreNomDeCompte>/vimubuntu
```
 Les variables <VotreNomDeCompte> correspond au nom de votre compte sur Docker Hub.

En retour de la commande, Docker nous donne un identifiant :
```sh
sha256:f22ba10fe40b9d87fb51dfaf7a0e4d5e863e6a3eb6edb1b2feb0516f82ad3443
```
C'est l'identifiant de l'image que nous avons construite à partir des changements effectués dans le conteneur.

Pour vérifier que nous avons bien créé une nouvelle image, supprimons notre ancien conteneur.
```sh
dev $ docker container rm vimubuntu
```
Listons les images pour essayer de retrouver la nôtre:
```sh
$ docker image ls
REPOSITORY                     TAG       IMAGE ID            CREATED             SIZE
<VotreNomDeCompte>/vimubuntu   latest    fe12ceb6e74c        8 minutes ago       186 MB
```
Notre image est bien listée comme <VotreNomDeCompte>/vimubuntu (où les variables sont remplacées par les valeurs correspondantes à votre environnement). Nous pouvons maintenant l’envoyer au registre avec la commande suivante :
```sh
dev $ docker image push <VotreNomDeCompte>/vimubuntu
```
 Relançons un autre conteneur grâce à cette image :
```sh
dev $ docker container run -i -t <VotreNomDeCompte>/vimubuntu /bin/bash
```
Tapez la commande vim dans le shell. On remarque que vim est déjà installé sur le conteneur. Nous avons sauvegardé nos changements avec succès!

Vous pouvez quitter vim en tapant sur la touche ESC puis en tapant :q! puis sortir du conteneur en lançant la commande exit.
## 8- Création d'une image à l'aide d'un Dockerfile

Nous savons que deux méthodes sont possibles pour construire une image. La deuxième, que nous allons voir maintenant, est la méthode recommandée. Nous allons décrire comment réaliser un petit fichier permettant de construire une image à l'aide d'une suite d'instructions. Ce fichier s'appelle le Dockerfile.

Nous allons créer notre propre application, et cette fois-ci à l'aide d'un Dockerfile !

Veillez à quitter le conteneur dans lequel vous êtes. Puis préparez à travailler dans le répertoire de travail `~/VotreNom/app/` Nous allons être amenés à créer plusieurs fichiers dans ce répertoire :

Dans le fichier `app.py`, copiez le code suivant (attention à l’indentation) :
```py
# app/app.py
from flask import Flask
from http import HTTPStatus
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Docker!\n"

@app.errorhandler(HTTPException)
def handle_http_exception(exception: HTTPException):
    return exception.description, exception.code

@app.errorhandler(Exception)
def handle_exception(exception: Exception):
    return HTTPStatus.INTERNAL_SERVER_ERROR.description, HTTPStatus.INTERNAL_SERVER_ERROR
```

Ce code représente une application que l’on fera évoluer pour montrer toutes les capacités de Kubernetes. Il utilise un framework Python qui s’appelle `Flask`. Il nous permettra d'ajouter des fonctionnalités de façon assez simple, sans avoir besoin de rajouter beaucoup de code.

Pour l’instant l’application est très basique : elle répond sur la route racine `/` par `Hello Docker` et un code `HTTP 200`.

Pour s'exécuter, l'application nécessite des modules comme `Flask`. On utilisera aussi `gunicorn`, un serveur Python WSGI HTTP. Nous allons mettre toutes ces dépendances dans un fichier de prérequis standard en Python : `requirements.txt`. Dans ce fichier, mettez le contenu suivant :
```sh
# app/requirements.txt
Flask==2.0.3
gunicorn==19.9.0
Jinja2==3.1.1
```

Éditez également le fichier `config.py` avec le contenu suivant :
```py
# app/config.py
import os

bind = "0.0.0.0:8000"
worker_class = "gthread"
threads = int(os.getenv('GUNICORN_CORES','1')) * 2 + 1
```
Utilisez l'éditeur vim pour créer un fichier Dockerfile. Dans ce fichier, ajoutez le contenu suivant :

```Dockerfile
# app/Dockerfile
FROM python:3.7-alpine
LABEL maintainer="<Nom> <Prénom>"

# Create and change working directory
WORKDIR /app

# Add application requirements
COPY requirements.txt .

# Install requirements
RUN pip install -r requirements.txt

# Add application
COPY app.py .
COPY config.py .

# Create a specific user to run the Python application
RUN adduser -D my-user -u 1000
USER 1000

# Launch application
ENTRYPOINT ["gunicorn"]
CMD ["--config", "config.py", "app:app"]
```

Prenons le temps d'étudier ces commandes :

`FROM` est une directive permettant de qualifier l'image de base utilisée. Pas de mystère, on utilise une version d’AlpineLinux.

`LABEL` est une directive qui permet d’ajouter des metadata à l’image. Ici, cela nous permet de définir la personne maintenant l'image. **Remplacez par vos nom et prénoms**.

`RUN` est l'une des directives principales d'un Dockerfile et permet d'exécuter des instructions qui vont permettre de construire notre image. Ici, on installe les dépendances de notre application avec pip.

`COPY` est une directive qui permet d’ajouter des fichiers provenant de notre host dans une image. Une option au comportement similaire est ADD, mais elle n’est pas conseillée vu son caractère “automagique”. Elle décompresse des fichiers et marche avec n’importe quel type de ressource, ce qui rend difficile de prévoir son comportement...

`WORKDIR` est une directive qui permet de changer le working directory par défaut de notre image.

`USER` est une directive permettant de spécifier quel sera l’utilisateur courant. Il est important de spécifier un utilisateur non-root (avec un uid supérieur ou égal à 1000) pour des raisons de sécurité.

`ENTRYPOINT` et `CMD` permettent de définir la commande qui est lancée au démarrage de notre conteneur. On parlera des différences entre ces deux directives plus tard dans la formation.

Maintenant que nous avons créé notre fichier Dockerfile, il est temps de créer notre fichier docker-compose qui va nous simplifier la construction et le lancement de notre application. Nous allons créer un fichier `docker-compose.yml` (toujours dans  `~/VotreNom/app/`) avec le contenu suivant :
```yaml
# app/docker-compose.yml
---
version: '3'
services:
  app:
    build: .
    image: "VotreNom/app:v0.1"
    ports:
    - "8000:8000"
```

Nous allons à présent utiliser `docker compose` une première fois pour construire notre image :
```
dev $ docker compose build
Building app
Step 1/11 : FROM python:3.7-alpine
 ---> c02a3409ee5b
Step 2/11 : LABEL maintainer="<Nom> <Prénom>"
 ---> Using cache
 ---> a2e9b278045e
Step 3/11 : WORKDIR /app
 ---> Running in 3ce50832d967
Removing intermediate container 3ce50832d967
 ---> d07cfb69c642
Step 4/11 : COPY requirements.txt .
 ---> fb52dbeb058f
Step 5/11 : RUN pip install -r requirements.txt
 ---> Running in e70df4a68b46
Collecting Flask==1.0.2 (from -r requirements.txt (line 2))
[]
Installing collected packages: MarkupSafe, Jinja2, itsdangerous, click, Werkzeug, Flask, gunicorn
Successfully installed Flask-1.0.2 Jinja2-2.10 MarkupSafe-1.1.0 Werkzeug-0.14.1 click-7.0 gunicorn-19.9.0 itsdangerous-1.1.0
Removing intermediate container e70df4a68b46
 ---> f1dac0f790ee
Step 6/11 : COPY app.py .
 ---> 597e9dfdca20
Step 7/11 : COPY config.py .
 ---> dc54fc18a0a7
Step 8/11 : RUN adduser -D my-user -u 1000
 ---> Running in 42050e09c8b6
Removing intermediate container 42050e09c8b6
 ---> f4c50ec0ccb9
Step 9/11 : USER 1000
 ---> Running in dc252f7b047c
Removing intermediate container dc252f7b047c
 ---> 06b4274f9bae
Step 10/11 : ENTRYPOINT ["gunicorn"]
 ---> Running in 6027d960cfdf
Removing intermediate container 6027d960cfdf
 ---> 39c5abb4193d
Step 11/11 : CMD ["--config", "config.py", "app:app"]
 ---> Running in b9260f53b0a6
Removing intermediate container b9260f53b0a6
 ---> 31920ffe5099
Successfully built 31920ffe5099
Successfully tagged 261407191094.dkr.ecr.eu-west-3.amazonaws.com/dalk/app:v0.1
```
Cette commande utilise `docker compose`, mais en réalité, elle ne fait qu’encapsulser pour vous un appel à la commande `docker image build` :
```sh
dev $ docker image build -t=<VotreNomDeCompte>/app:v0.1 .
```
Cette commande ordonne à Docker de créer une nouvelle image en respectant les instructions contenues dans le Dockerfile que nous venons de créer. Nous affectons également un nom à notre nouvelle image afin de pouvoir l'identifier avec le flag -t. Nous pouvons optionnellement affecter un **tag** de version différent de `latest` en ajoutant “:” puis le nom de la version à la suite du nom de l’image (`v0.1` dans notre cas).

L'exécution des instructions avec la commande `RUN` suit un workflow très particulier :
1. Docker lance un conteneur à partir de l'image de base
2. Une instruction est exécutée, par exemple `pip install -r requirements.txt` et un changement est effectué dans le conteneur.
3. Docker déclenche l'équivalent d'un `docker container commit` afin de créer une nouvelle couche de fichiers. Puis il crée une nouvelle image à partir de cette couche par dessus les couches précédentes.
4. Docker lance un nouveau conteneur grâce à cette nouvelle image.
5. On répète jusqu'à ce que toutes les commandes `RUN` soient exécutées et que l'on soit en possession d'une image finale à utiliser.

On peut remarquer que l’ID de l’image construite nous est communiqué en fin de build : `31920ffe5099` ici.

On peut remonter dans les actions effectuées pour la construction de notre image avec la commande `docker image history`.
```
dev $ docker image history 31920ffe5099
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
31920ffe5099        4 minutes ago       /bin/sh -c #(nop)  CMD ["--config" "config.p…   0B
39c5abb4193d        4 minutes ago       /bin/sh -c #(nop)  ENTRYPOINT ["gunicorn"]      0B
06b4274f9bae        4 minutes ago       /bin/sh -c #(nop)  USER 1000                    0B
f4c50ec0ccb9        4 minutes ago       /bin/sh -c adduser -D my-user -u 1000           4.82kB
dc54fc18a0a7        4 minutes ago       /bin/sh -c #(nop) COPY file:3e50f9b4a355f9de…   138B
597e9dfdca20        4 minutes ago       /bin/sh -c #(nop) COPY file:f2123582ae0ff603…   955B
f1dac0f790ee        4 minutes ago       /bin/sh -c pip install -r requirements.txt      11.3MB
fb52dbeb058f        4 minutes ago       /bin/sh -c #(nop) COPY file:70c56863bd832380…   63B
d07cfb69c642        4 minutes ago       /bin/sh -c #(nop) WORKDIR /app                  0B
a2e9b278045e        28 minutes ago      /bin/sh -c #(nop)  LABEL maintainer=<Nom> <P…   0B
c02a3409ee5b        6 hours ago         /bin/sh -c #(nop)  CMD ["python3"]              0B
<missing>           6 hours ago         /bin/sh -c set -ex;   wget -O get-pip.py 'ht…   6.04MB
<missing>           6 hours ago         /bin/sh -c #(nop)  ENV PYTHON_PIP_VERSION=19…   0B
<missing>           2 weeks ago         /bin/sh -c cd /usr/local/bin  && ln -s idle3…   32B
<missing>           2 weeks ago         /bin/sh -c set -ex  && apk add --no-cache --…   74.6MB
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV PYTHON_VERSION=3.7.2     0B
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV GPG_KEY=0D96DF4D4110E…   0B
<missing>           2 weeks ago         /bin/sh -c apk add --no-cache ca-certificates   551kB
<missing>           2 weeks ago         /bin/sh -c #(nop)  ENV LANG=C.UTF-8             0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  ENV PATH=/usr/local/bin:/…   0B
<missing>           3 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           3 weeks ago         /bin/sh -c #(nop) ADD file:2a1fc9351afe35698…   5.53MB
```
Nous pouvons également utiliser l'utilitaire *dive* pour avoir une vue plus fine des actions et leurs résultats par couches via la commande: `dive 31920ffe5099`.

Nous pouvons maintenant voir la totalité des commandes lancées lors de l'exécution de la
commande `docker image build`. Les instructions exécutées sont triées de la plus récente à la plus ancienne. Seule les 8 premières nous intéressent car ce sont les commandes produites par notre Dockerfile (les autres concernent les couches inférieures de l'image alpine). Chacune des instructions du Dockerfile produit bien une couche de fichiers puis une image avec un ID que Docker réutilise pour construire la couche suivante.

Il est intéressant de noter qu'il existe une limitation importante à Docker ([ou plutôt du système de fichiers dit union fs : Overlay2](https://github.com/docker/docker.github.io/issues/8230#issuecomment-468630187)) : **Nous sommes limités à 127 couches de fichiers !**

Si nous créons un Dockerfile et que nous atteignons la limite de 127 couches, nous ne pouvons plus construire par-dessus. Bien qu'en pratique cette limite est difficilement atteignable, il est important de l'avoir en tête afin de factoriser au maximum les instructions dans notre Dockerfile.

Un avantage de la factorisation à lieu lors de la suppression des fichiers. En effet, comme dans Git, la suppression d’un fichier lors d’un commit ne le supprime pas de l’historique (la couche précédente dans notre cas). Il est donc recommandé de procéder aux tâches de purge dans le mêmes lignes que celle qui font des installations. Exemple :
```Dockerfile
RUN apt-get -y update && apt-get install -y \
  aufs-tools \
  automake \
  build-essential \
  curl \
  dpkg-sig \
  libcap-dev \
  libsqlite3-dev \
  mercurial \
  reprepo \
  ruby1.9.1 \
  ruby1.9.1-dev \
  s3cmd=1.1.* \
  && rm -rf /var/lib/apt/lists/*
```
Notez bien que toutes les commandes sont enchaînées dans une seule instruction `RUN` du `Dockerfile` via des `&&` et se termine par un `rm`. Une seule couche est créée et les fichiers temporaires sont supprimés avant le commit.

Il est à présent temps de démarrez l’application. Pour ce faire, exécutez la commande suivante :
```sh
dev $ docker compose up
Creating network "app_default" with the default driver
Creating app_app_1 ... done
Attaching to app_app_1
app_1 | [2019-01-08 15:53:33] [1] [INFO] Starting gunicorn 19.9.0
app_1 | [2019-01-08 15:53:33] [1] [INFO] Listening at: http://0.0.0.0:8000 (1)
app_1 | [2019-01-08 15:53:33] [1] [INFO] Using worker: gthread
app_1 | [2019-01-08 15:53:33] [7] [INFO] Booting worker with pid: 7
```

Ouvrez une autre session shell et essayez de requêter l’application :
```sh
dev $ curl localhost:8000 -v
...
>
< HTTP/1.1 200 OK
< Server: gunicorn/19.9.0
< Date: Fri, 06 Jul 2018 15:27:33 GMT
< Connection: close
< Content-Type: text/html; charset=utf-8
< Content-Length: 12
<
* Closing connection 0
Hello Docker!
```
On voit que l’application répond avec un status code HTTP 200 et le message “Hello Docker!”.
## 9- Transférer notre image sur le registre

Maintenant que nous avons notre image locale, le but est de la transférer sur le **registre central** afin que tout le monde puisse en profiter en appelant la commande `docker image pull`. Transférons notre image **app** sur le **registre** que nous avons utilisé plus tôt lors de ce TP. Pour effectuer cette action, nous devons utiliser la commande `docker compose push`.
```
dev $ docker compose push
Pushing app (<VotreNomDeCompte>/app:v0.1)...
The push refers to a repository [<VotreNomDeCompte>/app]
fe248af17b1b: Pushed
5a54a6dcff20: Pushed
50f589ab1c5f: Pushed
75fed4290ceb: Pushed
6795dbd93463: Pushed
e2986b5e7ba2: Pushed
beefb6beb20f: Pushed
df64d3292fd6: Pushed
v0.1: digest: sha256:683e3fa0b802b4216c80434ae688124b48cec58bfba628bd56944c9e5a614c78 size: 1993
```
Cette commande utilise `docker compose`, mais en réalité, elle ne fait qu’encapsulser pour vous un appel à la commande `docker image push` :
```sh
dev $ docker image push <VotreNomDeCompte>/app:v0.1
```
En analysant la trace, on remarque que Docker détecte que nous voulons pousser notre image dans le dépôt **app** situé dans le registre accessible à l'adresse `<VotreNomDeCompte>`.
Ensuite, Docker transfère une à une les couches qui forment notre image sur notre registre. Notre registre contient ainsi notre image, et s'il était public et accessible, n'importe qui pourrait appeler la commande `docker image pull` afin de télécharger l'image et ensuite appeler `docker container run` pour construire un conteneur à partir de cette image.
## 10- Nettoyage des conteneurs

Nous allons supprimer tous les conteneurs :
```sh
dev $ docker system prune
```
La commande `prune` nous permet de supprimer les ressources Docker qui ne sont plus utilisées dans notre machine. Dans ces ressources, on trouve les conteneurs arrêtés, les réseaux non-utilisés, les images sans nom (dangling), et les volumes.
## 11- Résumé des commandes

Bravo! Vous avez pris en main les commandes de base de Docker afin de créer et manipuler des images. Dans la prochaine section, nous allons voir quelques commandes supplémentaires pour créer un Dockerfile. En attendant voici un petit résumé des commandes que nous avons vues dans cette section:

`docker image ls` Liste les images que l'on possède en local.
`docker search` Cherche une image dans le registre central (Docker Hub)
`docker image pull` Télécharge une image depuis le Docker Hub ou un registre privé.

`docker container commit` Sauvegarde les changements réalisés dans une image (installation d'un paquet, etc.) et crée une image réutilisable.

`Dockerfile` Fichier permettant de créer une image en décrivant une suite d'instructions.
- `FROM` L'image de base que l'on doit utiliser pour la construction de notre nouvelle image.
- `MAINTAINER` Le créateur/mainteneur de l'image.
- `RUN` Exécute une commande shell et crée une nouvelle couche de système de fichiers en fonctions des changements effectués.

`docker image build` Construit une image à partir d'un Dockerfile.
- `--tag / -t` Nom de l'image à construire.

`docker image history` Donne des informations sur la hiérarchie de couches de 	notre conteneur.

`docker image push` Transfère l'image sur le registre central afin de la partager 	avec la communauté ou d'autres développeurs.


## 12- Question bonus

>**Question pour les plus rapides**
>- Dans les faits, nous n’utilisons jamais la commande `docker container commit`. Sauriez-vous dire pourquoi son usage n’est pas recommandé ?
