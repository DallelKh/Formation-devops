# Docker / Kubernetes - TP1 : Images, conteneurs et volumes

> **Objectifs du TP** :
> - Se familiariser avec les concepts d’images et de conteneurs
> - Manipuler les commandes de base de Docker : Créer un conteneur et le manipuler
> - Comprendre le fonctionnement et la manipulation des volumes
>
> **Niveau de difficulté** : Débutant

## 1- Premières commandes Docker

Nous allons commencer par prendre en main l'outil en ligne de commande permettant de créer et gérer des conteneurs et des images. Vérifions tout d'abord que le démon Docker s'exécute bel et bien sur notre machine virtuelle :

```sh
dev $ docker info && docker version
```

Normalement si tout se passe bien vous devriez voir apparaitre une sortie de la forme suivante:

```
Containers: 0
 Running: 0
 Paused: 0
 Stopped: 0
Images: 0
Server Version: 17.05.0-ce
Storage Driver: overlay2
 Backing Filesystem: extfs
 Supports d_type: true
 Native Overlay Diff: false
Logging Driver: json-file
Cgroup Driver: cgroupfs
Plugins:
 Volume: local
 Network: bridge host macvlan null overlay
Swarm: inactive
Runtimes: runc
Default Runtime: runc
Init Binary: docker-init
containerd version: 9048e5e50717ea4497b757314bad98ea3763c145
runc version: 9c2d8d184e5da67c95d601382adf14862e4f2228
init version: 949e6fa
Security Options:
 apparmor
 seccomp
  Profile: default
Kernel Version: 4.4.0-34-generic
Operating System: Ubuntu 16.04.1 LTS
OSType: linux
Architecture: x86_64
CPUs: 2
Total Memory: 3.858GiB
Name: ip-10-0-0-37
ID: D7YT:PKOS:KGTQ:Q32D:JVA4:6O7W:FV2W:DVGW:VC3Q:YKWQ:W4SA:P5RA
Docker Root Dir: /var/lib/docker
Debug Mode (client): false
Debug Mode (server): false
Registry: https://index.docker.io/v1/
Experimental: false
Insecure Registries:
 127.0.0.0/8
Live Restore Enabled: false

WARNING: No swap limit support
Client:
 Version:      17.05.0-ce
 API version:  1.29
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:20:50 2017
 OS/Arch:      linux/amd64

Server:
 Version:      17.05.0-ce
 API version:  1.29 (minimum version 1.12)
 Go version:   go1.7.5
 Git commit:   89658be
 Built:        Thu May  4 22:20:50 2017
 OS/Arch:      linux/amd64
 Experimental: false
```

**Analysons cette sortie** :

Elle nous dit qu'il y a actuellement 0 conteneur Docker qui s'exécute sur la machine et que nous ne possédons aucune image à l’heure actuelle. Nous reviendrons plus tard sur ce qu'est une image et comment il est possible de les utiliser pour construire des conteneurs.
La sortie nous montre aussi que le driver de stockage utilisé est overlayfs. Nous y reviendrons également plus tard mais c'est un détail important concernant la gestion et la construction des images Docker. Enfin, nous obtenons des informations sur la version du noyau Linux utilisée ainsi que la version de Docker utilisée.

Allons plus loin en créant notre premier conteneur !

## 2- Création d'un premier conteneur Docker

Très bien ! Créons désormais notre premier conteneur en lançant une commande avec Docker :

```sh
dev $ docker container run -i -t ubuntu /bin/bash
```

Que nous dit cette ligne ?
- On ordonne à Docker de lancer un conteneur grâce à la commande `docker container run`
- Le flag `-i` demande à Docker de laisser ouvert le flux d'entrée standard du conteneur pour pouvoir lui passer des commandes par ce flux.
- Le flag `-t` ordonne à Docker d'assigner un tty au conteneur que l'on crée. Cela va nous permettre de passer des commandes à notre conteneur (en lui assignant un shell).
- On veut utiliser une **image de base** qui correspond à une `ubuntu`. Notre conteneur disposera ainsi d'une structure de dossier identique à celle de la dernière distribution Ubuntu.
- Finalement, on passe la commande que l'on veut exécuter sur le conteneur. Ici on choisit d'exécuter `/bin/bash` afin d'avoir accès à un shell et pouvoir taper des commandes.

Si vous avez l'œil aguerri, vous avez dû remarquer que le lancement de cette commande crée un flot d'événements en sortie :

```sh
Unable to find image 'ubuntu:latest' locally
latest: Pulling from library/ubuntu
6bbedd9b76a4: Pull complete
fc19d60a83f1: Pull complete
de413bb911fd: Pull complete
2879a7ad3144: Pull complete
668604fde02e: Pull complete
Digest: sha256:2d44ae143feeb36f4c898d32ed2ab2dffeb3a573d2d8928646dfc9cb7deb1315
Status: Downloaded newer image for ubuntu:latest
```

Docker ne trouve pas l'image de base Ubuntu sur le système alors il va la télécharger depuis un **registre central**, le **Docker Hub** (https://hub.docker.com/). Nous reviendrons plus tard sur la notion de système de fichiers en couches mais il faut savoir que chaque ligne avec un identifiant et la note "Download complete" correspond à une couche qui permet de construire l'image de base.

**Vous devriez maintenant avoir accès à un shell à l'intérieur de votre conteneur.**
```sh
root@a68455defe90:/#
```
Il est important de noter que vous êtes désormais dans un environnement totalement isolé de votre système hôte. Rentrons donc une commande afin d'installer un éditeur de texte bien connu : VIM.
```sh
root@a68455defe90:/# apt-get update && apt-get install vim
[...]
Fetched 24.5 MB in 2s (8330 kB/s)
Reading package lists... Done
Reading package lists... Done
Building dependency tree
[...]
The following NEW packages will be installed:
  file libexpat1 libgpm2 libmagic1 libmpdec2 libpython3.5 libpython3.5-minimal
  libpython3.5-stdlib libsqlite3-0 libssl1.0.0 mime-support vim vim-common vim-runtime
0 upgraded, 14 newly installed, 0 to remove and 15 not upgraded.
Need to get 12.2 MB of archives.
After this operation, 58.3 MB of additional disk space will be used.
Do you want to continue? [Y/n]
```
**Tapez Y (Yes) pour continuer l'installation.** Tout ce que vous venez d'effectuer en termes d'installation restera dans le conteneur. Si vous le détruisez, il ne restera plus rien !

Tiens, essayons de l’arrêter ! Tapez `exit` sur le terminal du conteneur afin d'en sortir. Vous revenez ainsi à votre shell classique. Tapons la commande suivante afin de lister les conteneurs.
```sh
dev $ docker container ps
```
La commande `docker container ps` permet de lister les conteneurs s'exécutant sur la machine. En tapant cette commande, vous remarquerez que notre conteneur n'y apparaît pas. Pas de panique, c'est tout à fait normal. En quittant notre conteneur, nous l'avons **stoppé**. Il existe toujours pour Docker cependant, tant qu'on ne lui dit pas explicitement de le supprimer (par la commande `docker container rm`), il restera disponible sur la machine.

En effet, un conteneur ne marche pas comme une machine virtuelle pour laquelle on doit expressément envoyer un signal de fin de tâche. **Un conteneur ne vit que pour le processus qu'il contient !** Imaginons donc que nous lançons un script (voué à se terminer) au sein d'un conteneur Docker. À la fin de ce script, les processus lancés par le script se termineront mais le conteneur également. Il n'y a aucun intérêt à garder en vie un conteneur n'hébergeant aucun processus tout comme l’on n’aurait aucun intérêt à garder en vie une machine virtuelle qui n'exécute aucun processus et ne ferait que gâcher des ressources.

**On voit ici toute la puissance de Docker, le fait d'exécuter des processus de manière isolée sans gâcher de ressources !**

Mais alors comment lister nos conteneurs morts ? C'est simple, tapez la commande suivante :
```sh
dev $ docker container ps -a
```
Vous remarquerez la similitude avec la commande `ps` de Linux. Le flag `-a` permet de lister l’intégralité des conteneurs, même ceux qui sont dans un état `terminated`. Vous devriez désormais voir votre conteneur dans la liste, essayez donc de retrouver son identifiant.

## 3- Démarrer un conteneur stoppé

Relançons un autre conteneur mais cette fois-ci avec une option supplémentaire permettant de lui donner un nom :
```sh
dev $ docker container run --name my_container -i -t ubuntu /bin/bash
```
Nous avons lancé un nouveau conteneur nommé **my_container**. Tapez `exit` une fois que vous êtes dedans.

On peut maintenant redémarrer le conteneur avec la commande suivante :
```sh
dev $ docker container start my_container
```
Si on relance la commande `docker container ps` (mais cette fois **sans le flag** `-a`), on peut voir que notre conteneur est toujours en vie et exécute de manière isolée un shell.

## 4- S'attacher à un conteneur

Le conteneur a redémarré avec les mêmes options que lors de son premier lancement. Une session interactive nous attend sur le conteneur. Pour y accéder, nous devons nous **attacher** au conteneur.
```sh
dev $ docker container attach my_container
```
Vous devriez voir apparaître un shell comme la première fois. Appuyez sur la touche « Entrée » si vous ne le voyez pas.
De la même manière que la commande précédente, on peut également passer l'id du conteneur.

Pour sortir du conteneur en mode attaché sur un shell, tapez simplement `exit`.

>- Est ce que le conteneur tourne encore ?
>- Que s'est-il passé ?

Relancez le conteneur et attachez-vous de nouveau
```sh
dev $ docker container start my_container
dev $ docker container attach my_container
```

Pour se détacher du conteneur ce coup-ci sans mettre fin au processus /bin/bash qui tourne, tapez `ctrl+p,ctrl+q`.
(Il sera préférable d'utiliser la console pour cette commande plutôt que le WebIDE)
Vérifiez que le conteneur tourne encore.

## 5- Créer un conteneur démon

En complément de lancer des conteneurs avec des sessions de shell interactive, on peut également lancer des conteneurs qui vont exécuter des programmes longs. Tentez donc de taper la commande suivante :
```sh
dev $ docker container run --name my_daemon -d ubuntu /bin/sh -c "while true; do echo hello world; sleep 1; done"
```
Nous utilisons la commande docker container run couplée au flag `-d` pour dire à Docker que l'on veut que le conteneur soit lancé en arrière plan.

Nous avons également passé une boucle while en commande shell afin de boucler indéfiniment jusqu'à ce que l'on stoppe le conteneur ou le processus de manière explicite.

Si nous tapons la commande `docker container ps`, nous devrions voir notre conteneur lancé en arrière plan avec un statut qui est **Up**.
```sh
CONTAINER ID  IMAGE   COMMAND                 CREATED             STATUS
613bdd4821ae  ubuntu  "/bin/sh -c 'while tr"  About a minute ago  Up...
```
## 6- Voir ce qu'il se passe dans notre conteneur !

Très bien. Allons maintenant jeter un œil sur ce qu'il se passe dans notre conteneur ! Attention cependant à **ne pas utiliser** `docker container attach`. Lancer cette commande nous permet de voir l'exécution de la boucle. Cependant, pour pouvoir sortir du mode **attaché**, il faudra utiliser Ctrl-C. En plus de stopper le conteneur, cela enverra un signal SIGTERM qui aura pour effet d’arrêter le processus. Une question se pose alors : comment voir des logs d'exécution de notre programme ?

Une autre commande existe, `docker container logs`, permettant de voir les logs produits par le conteneur sur la sortie standard **stdout**. Tapez donc la commande suivante :
```sh
dev $ docker container logs my_daemon
```
Vous devriez voir la sortie attendue :
```
hello world
hello world
hello world
hello world
hello world
[...]
```
Ici, la commande `docker container logs` affiche les dernières sorties (sur **stdout**) produites par le script exécuté dans notre conteneur. Cependant, il est également possible de voir ces sorties au fur et à mesure qu'elles sont produites de la même façon que `docker container attach` grâce à la commande suivante :
```sh
dev $ docker container logs -f my_daemon
```
On voit apparaître "hello world" au fur et à mesure et en tapant Ctrl-C, on sort de la commande `docker container logs` sans pour autant stopper le conteneur. Ce qui est pratique pour déboguer le programme dans un conteneur sans stopper son exécution (par exemple, analyser les stacktraces d'un programme Java).

Imaginons maintenant que l'on veuille les 20 dernières lignes de logs pour des questions de lisibilité (on veut simplement vérifier la bonne exécution d'un programme sans avoir la totalité des logs produits par `docker container logs`). Pour cela, on passe simplement un nouvel argument à `docker container logs`, l’option `--tail`, qui permet de récupérer les dernières logs produits. Par exemple :
```sh
dev $ docker container logs --tail 20 my_daemon
```
Nous retournera les 20 dernières lignes produite par notre script. Pratique!

## 7- Inspecter les processus s'exécutant dans un conteneur

Il serait pratique de connaître les processus qu'un conteneur exécute. Pour ce faire, il existe la commande `docker container top`.
```sh
dev $ docker container top my_daemon
```
Cette commande va lister tous les processus qui s'exécutent dans notre conteneur de la même manière que l'on utiliserait la commande top de Linux pour lister les processus.
```sh
UID   PID    PPID   C  STIME  TTY  TIME      CMD
root  16597  16581  0  15:03  ?    00:00:00  /bin/sh -c while true; d[...]
root  18857  16597  0  15:39  ?    00:00:00  sleep 1
```

## 8- Inspecter les informations d'un conteneur

On peut également inspecter une quantité d'informations pratique sur notre conteneur au format JSON en utilisant la commande `docker container inspect`.

```sh
dev $ docker container inspect my_daemon
[
  {
    "Id": "613bdd482[...]",
        "Created": "2016-11-16T14:03:34.006557847Z",
        "Path": "/bin/sh",
        "Args": [
            "-c",
            "while true; do echo hello world; sleep 1; done"
        ],
        "State": {
            "Status": "running",
            "Running": true,
            "Paused": false,
            "Restarting": false,
            "OOMKilled": false,
            "Dead": false,
            "Pid": 16597,
            "ExitCode": 0,
            "Error": "",
            "StartedAt": "2016-11-16T14:03:34.247842989Z",
            "FinishedAt": "0001-01-01T00:00:00Z"
        },
[...]
```
La commande `inspect` interroge le conteneur et retourne un grand nombre d’information comme les informations de configurations globales par exemple.

On peut également utiliser une syntaxe spéciale afin de ne requêter que les informations dont on a besoin. Par exemple, pour récupérer que l'adresse IP du conteneur, on utilise la commande suivante :
```sh
dev $ docker container inspect --format='{{.NetworkSettings.IPAddress}}' my_daemon
```
qui devrait afficher en retour l’adresse IP :
```sh
172.17.0.2
```
Si nécessaire, on peut donner plusieurs conteneurs en argument afin de retourner les informations de plusieurs conteneurs.

## 9- Stopper un conteneur

Maintenant que nous avons bien joué avec notre conteneur, il est temps de le stopper. Pas de mystère, on utilise la commande `docker container stop` pour cela.
```sh
dev $ docker container stop my_daemon
```
Facile !

## 10- Supprimer un conteneur pour de bon

Si vous voulez supprimer un conteneur pour de bon avec tout son contenu (afin qu'il ne soit même plus listé dans les conteneurs stoppés), vous pouvez utiliser la commande `docker container rm`.
```sh
dev $ docker container rm my_daemon
```
Il existe maintenant un moyen simple de supprimer tous les conteneurs qui sont déjà stoppés avec la commande `docker container prune`.
```sh
dev $ docker container prune
```
Il n'existe pas de moyen pratique de supprimer tous les conteneurs (par exemple un `docker container rm -a`). Cependant, on peut coupler les commandes de manière élégante pour arriver à nos fins.
```sh
$ docker container rm $(docker ps --filter 'status=exited' -a -q)
```
La commande appelle `docker container rm` en lui passant une liste d'identifiants de conteneurs récupérée à l'aide de la commande `docker container ps`.


## 11- Attacher un volume à notre conteneur

Par défaut, les conteneurs que nous créons génèrent des modifications dans leur environnement. Si on supprime le conteneur, il ne restera plus rien.

Pour conserver les modifications d'un conteneur, on peut attacher un volume de données.

Commençons par créer un nouveau volume : **volume_test** :
```sh
dev $ docker volume create volume_test
```
Maintenant que nous avons créé notre volume, nous allons monter ce volume sur un nouveau conteneur. Pour le faire, nous utilisons le flag `-v` (ou `--volume`) de la commande `docker container run` :
```sh
dev $ docker container run -i -t -v volume_test:/volume_test ubuntu /bin/bash
```
Notre conteneur a accès au volume **volume_test** et peut y écrire des données. Créons maintenant un fichier dans ce dossier **/volume_test** avec la commande suivante :
```
# touch /volume_test/testfile
```
Sortons du conteneur avec `exit` et tuons-le avec `docker container rm`.

Maintenant, créons un autre container avec **volume_test**. Nous pouvons voir que le fichier existe toujours dans le volume.

Au départ, l’objectif des volumes peut-être confus mais il prend tout son sens lorsque l'on commence à manipuler les conteneurs afin d’y appliquer des modifications. Retenez que toute écriture de données devant persister après la destruction du conteneur doit passer par un volume.

## 12- Résumé des commandes

Bravo ! Vous avez pris en main les commandes de base de Docker afin de créer et manipuler un **conteneur**. Le chemin est encore long mais résumons les commandes que nous avons vu ensemble dans cette première partie de TP :

`docker info` Liste les informations concernant le démon Docker de l'hôte.


`docker --version` Donne des informations sur la version du démon Docker.

`docker container run` Démarre un conteneur, avec :
- `-i` Laisse ouvert l'entrée standard du conteneur
- `-t` Assigne un pseudo-tty (pour session interactive avec shell)
- `-d` Sert à démarrer un conteneur démon.
- `--name` Sert à modifier le nom par défaut assigné au conteneur.
- `-v / --volume` Monte un dossier de la machine hôte sur le système de fichiers du conteneur.

`docker container ls` Liste les conteneurs démarrés.

`docker container ls -a` Liste tous les conteneurs (même ceux qui sont stoppés).

`docker container start` Démarre ou redémarre un conteneur.

`docker container attach` S'attache à la session interactive du conteneur ou observe ses logs (mais Ctrl-C tue le processus/conteneur).

`docker container logs`	Affiche les logs et la sortie standard du conteneur (tous), avec :
- `-f` Affiche le contenu et ce qui est produit en live.
- `--tail X` Affiche les X dernières entrées de la sortie standard.

`docker container  top`	Liste les processus qui s'exécutent dans un conteneur.

`docker container stop` Stoppe le conteneur (il reste redémarrable).

`docker container rm` Supprime le conteneur (et son contenu) définitivement.

`docker container prune` Supprime tous les conteneurs (et leur contenu) stoppés définitivement.
