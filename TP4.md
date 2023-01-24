# Docker / Kubernetes - TP4 : Contextes, namespaces et premières exécutions
> **Objectifs du TP** :
>- Manipuler les contextes `kubectl`
>- Manipuler la labellisation des objets
>- Apprendre à écrire des sélecteurs
>- Déployer une application dans Kubernetes
>- Créer et manipuler des ***configmap***
>- Créer des ressources directement via des fichiers YAML
> **Niveau de difficulté** : Débutant

>>>Notez bien sur le TP, les resultats des commandes sont relatifs à l'environement de chaque utilisateur, utilisez les variables de votre popre environement (nom du cluster, nom de node, credentials de docker...)...

Pour les questions sufixées par "[AnswerX]", merci de d'inscrire la réponse en remplaçant "<fix me>" dans le fichier "answers/answers_tpX" au niveau de la ligne correspondante '"AnswerX": "<fix me>"'

## 1- Contexte d’utilisation

Lors du précédent TP, nous avons pu utiliser notre client `kubectl` et communiquer avec notre cluster, mais nous n’avons pas réellement compris par quelle magie cela a été possible.

Prenons quelques minutes pour regarder ça en détail. Nous allons commencer par chercher le fichier de configuration qui contient les informations pour se connecter au cluster Kubernetes. Si l’on part à la recherche de la variable d’environnement `$KUBECONFIG`, notre espoir est vite douché :

```sh
dev $ echo $KUBECONFIG

dev $
```

Pas de chance. Nous en concluons donc que `kubectl` se base sur le chemin par défaut pour trouver le fichier de configuration.

> **Questions**
> - Où se trouve le fichier de configuration ?
> - Dans quel format est-il écrit ?
> - Combien de contextes sont décrits dans le fichier de configuration ? [Answer1]

Même s’il y a plusieurs contextes, il est possible de demander à `kubectl` de n’afficher que celui qui est actif :

```sh
dev $ kubectl config view --minify
[...]
```
> **Questions**
>- Quelle est l’adresse IP sur laquelle se connecte notre client `kubectl` ? [Answer2]
>- Dans quel ***namespace*** travaille votre contexte ? [Answer3]
>- Quel est le mode d’authentification utilisé pour se connecter au cluster ?

## 2- Labellisation et sélecteurs

Nous allons créer des objets pour pouvoir manipuler des labels qui y sont associés. Dans un premier temps, peu importe leur utilité, ce qui nous intéresse n’est pas à quoi ils servent, mais bien la capacité de poser des labels sur n’importe quel type d’objets et de pourvoir les requêter par la suite. Dans cet exercice, nous allons simplement manipuler des ***configmap*** (***cm***).

```sh
dev $ kubectl create cm cm1
dev $ kubectl create cm cm2
dev $ kubectl create cm cm3
dev $ kubectl create cm cm4
dev $ kubectl create cm cm5
```

Nous allons à présent labelliser nos nouvelles ***cm*** :

```sh
dev $ kubectl label cm/cm1 version=1.0 country=fr
dev $ kubectl label cm/cm2 version=1.0 country=uk
dev $ kubectl label cm/cm3 version=1.1 country=de
dev $ kubectl label cm/cm4 version=1.1 country=fr
```

Le premier label `version` permet de simplement identifier les ***cm*** par leur version. Ils peuvent être retrouvés simplement grâce à un sélecteur :

```sh
dev $ kubectl get cm -l version=1.0
NAME   DATA   AGE
cm1    0      107s
cm2    0      100s
```

On voit que les autres ***cm*** sont absents de la liste, qu’ils aient le label `version` à une autre valeur que `1.0` ou que ce label soit absent.

> **Exercice**
> - Écrire un filtre permettant de ne sélectionner que les cm avec `country=fr`
> - Écrire un filtre permettant de ne sélectionner que les cm avec `version=1.1` **ET** `country=fr`
> - Écrire un filtre permettant de ne sélectionner que les cm qui n’ont pas le label `version` positionné

## 3- Labellisation des nœuds

Nous avons vu que les labels peuvent s’appliquer à n’importe quel type de ressources. Il faut savoir qu’il existe des ressources que l’on connaît déjà qui dispose par défaut de labels :

```sh
dev $ kubectl get no/k8s-training-dkusr-n1 -o template --template='{{range $k,$v:=.metadata.labels}}{{$k}}={{$v}}{{"\n"}}{{end}}'
beta.kubernetes.io/arch=amd64
beta.kubernetes.io/os=linux
kubernetes.io/hostname=k8s-training-dkusr-n1
```

Équivalent avec une sortie au format JSON et un parse avec l’utilitaire `jq` :

```sh
dev $ kubectl get no/k8s-training-dkusr-n1 -o json | jq .metadata.labels
{
  "beta.kubernetes.io/arch": "amd64",
  "beta.kubernetes.io/os": "linux",
  "kubernetes.io/hostname": "k8s-training-dkusr-n1",
}
```

Dernier moyen de voir les labels des objets : utiliser l’option `--show-labels` lors de l’utilisation de la commande `kubectl get`.

```sh
dev $ kubectl get no k8s-training-dkusr-n1 --show-labels
NAME                    STATUS   ROLES    AGE     VERSION   LABELS
k8s-training-dkusr-n1   Ready    <none>   4h17m   v1.12.1   beta.kubernetes.io/arch=amd64,beta.kubernetes.io/os=linux,kubernetes.io/hostname=k8s-training-dkusr-n1
```

Labelliser les nœuds est une opération classique que les administrateurs utilisent car elle permet de dédier des nœuds pour ne porter que certaines ressources et donc certains conteneurs spécifiques.

À titre d’exemple, il arrive de mettre des labels à certains nœuds d’un cluster K8s en DMZ pour qu’ils portent des types d’objets en particulier. Par exemple, cela peut servir à spécifier le type de disque dont dispose le nœud.

Un autre usage de la labellisation des nœuds intervient pour des clusters hybrides, c’est à dire s’il existe des nœuds de différentes architectures au sein d’un même cluster. Les conteneurs Docker n’étant pas portables d'une architecture à l'autre, il faut rigoureusement respecter l’adéquation entre l’architecture des conteneurs et celle des nœuds qui les exécutent.

> **Exercice**
>
> Après avoir trouvé les labels présents sur les nœuds, écrire un sélecteur pour lister tous les nœuds qui n’ont pas le **rôle** **`master`**

## 4- Première exécution
Maintenant que tout est en place, nous allons commencer à réellement lancer des conteneurs !!

La commande magique `kubectl run` est là pour nous simplifier la vie :

```
dev $ kubectl run -ti --rm --restart=Never --image=busybox busybox
If you don't see a command prompt, try pressing enter.
/ # cat /etc/resolv.conf
nameserver 10.0.0.10
search dalk.svc.cluster.local svc.cluster.local cluster.local
options ndots:5
/ # exit
```

Même si cela paraît encore un peu mystérieux, noter la ligne :

```sh
search dalk.svc.cluster.local [...]
```

Vérifiez avec votre voisin de formation qu’il a bien une valeur différente de la vôtre.

Visiblement, il semblerait que le DNS ait un paramétrage différent d’un ***namespace*** à l’autre…

> **Question**
>
> Sauriez-vous expliquer pourquoi vous trouvez des valeurs différentes ?

Nous venons de lancer un ***pod***, mais à l’instar d’une commande `docker container run --rm`, il n’avait pas vocation à persister.

D’ailleurs, si vous lancez la commande suivante, vous aurez la preuve que ce pod n’a eu une durée de vie très courte, puisque l’on n’en trouve plus la trace :

```sh
dev $ kubectl get po
No resources found.
```

## 5- Lancement de l’application kubernetes-application

Après avoir déployé nos premières ressources dans Kubernetes, il est temps de déployer notre kubernetes-application initiée dans le TP2.

Comme vous l’avez vu tout à l’heure en déployant votre premier pod dans Kubernetes via la commande `kubectl run`, vous devez indiquer l’image du conteneur que vous souhaitez déployer. En l'occurrence, il s’agissait de l’image `busybox` que Kubernetes va aller chercher par défaut sur le docker hub.

Dans le TP2, nous avons construit, et poussé notre image dans une registry distante. C’est depuis cette registry que Kubernetes récupérera notre application afin de la déployer.

Nous voilà enfin prêts à déployer notre application. Comme vous le savez désormais, déployer une application dans Kubernetes nécessite la création d’au moins 4 ressources :
1. Un ***deployment***
1. Un ***replicaset*** (créé par le ***deployment***)
1. Un ***pod*** (créé par le ***replicaset***)
1. Un ***service***

Les ressources Kubernetes peuvent être créées via la commande `kubectl run`, mais cela peut complexifier le suivi du cycle de vie de notre application. Une autre approche consiste à décrire l’attendu de nos ressources dans un fichier yaml ou json. C’est cette approche que nous allons privilégier.

Vous trouverez ci-dessous un exemple d’une ressource de type ***deployment***. Créez un fichier `deployment-kubernetes-app.yaml` (dans le dossier  `~/dalk/deployment/`) et copiez-y le code suivant :

```yaml
# deployment/deployment-kubernetes-app.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-app
  labels:
    app: kubernetes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubernetes-app
  template:
    metadata:
      labels:
        app: kubernetes-app
    spec:
      containers:
      - name: kubernetes-app
        image: 261407191094.dkr.ecr.eu-west-3.amazonaws.com/dalk/app:v0.1
        ports:
        - containerPort: 8000
          name: http
```

Pour que kubernetes puisse interpréter ce fichier, nous allons lui passer en utilisant le client `kubectl` :

```sh
dev $ kubectl apply -f deployment-kubernetes-app.yaml
```

> **Question**
>
> Quelles ressources ont été créées suite à cette commande ?


Une bonne partie de nos ressources ont été créées, mais nous constatons un problème : le pod ne parvient pas à être déployé.
```sh
NAME                              READY     STATUS             RESTARTS   AGE
kubernetes-app-5dbd6fd97f-9gmd4   0/1       ImagePullBackOff   0          13s
```

Investiguons un instant. Les commandes `kubectl get events` et `kubectl describe po` vont nous aider à comprendre ce qu’il se passe.

> **Exercice**
>
> Essayez de comprendre la raison pour laquelle notre application ne peut pas être déployée.

Vous l’aurez compris, nous avons un problème d’authentification lors de la récupération de notre image dans la registry privée. Nous allons devoir donner les credentials à notre ***deployment***. Pour cela, nous allons passer par une ressource de type ***secret***. il faut créér ce secret. Il porte le nom de **`regsec`** : (chercher comment creer le secret), une fois créé, s'assurer qu'il existe

```sh
dev $ kubectl get secret regsec
NAME     TYPE                             DATA   AGE
regsec   kubernetes.io/dockerconfigjson   1      28m
```

Vous remarquerez que, pour le moment, notre ***deployment*** ne fait pas référence à ce ***secret***, ce qui explique l’erreur. Pour y remedier, nous allons modifier notre fichier `deployment-kubernetes-app.yaml`.

```yaml
# deployment/deployment-kubernetes-app.yaml
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: kubernetes-app
  labels:
    app: kubernetes-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: kubernetes-app
  template:
    metadata:
      labels:
        app: kubernetes-app
    spec:
      containers:
      - name: kubernetes-app
        image: 261407191094.dkr.ecr.eu-west-3.amazonaws.com/dalk/app:v0.1
        ports:
        - containerPort: 8000
          name: http
      imagePullSecrets: # <== Ajout
      - name: regsec    # <== Ajout
```

Ceci fait, utilisez une nouvelle fois la commande `kubectl apply -f` pour l’envoyer à votre cluster Kubernetes. Cette commande est idempotente, ce qui signifie que seuls les changements sont pris en comptes, tout ne sera pas écrasé.

```sh
dev $ kubectl apply -f deployment-kubernetes-app.yaml
```

C’est fait, notre application est déployée. Rendons-la disponible en l’exposant derrière un ***service***. Comme la machine de dev sur laquelle nous travaillons n’est pas dans le cluster Kubernetes, il faut préciser que nous allons créer un type de service particulier. Pour cela, nous allons nous aider de la commande suivante :

```sh
dev $ kubectl expose deploy/kubernetes-app --type=NodePort
```

Nous avons laissé le cluster choisir pour nous un `nodePort`, que l’on peut récupérer avec la commande suivante :

```sh
dev $ kubectl get svc
NAME           TYPE       CLUSTER-IP     EXTERNAL-IP   PORT(S)          AGE
kubernetes-app NodePort   10.97.54.113   <none>        8000:32328/TCP   5s
```

Dans cet exemple, le `nodePort` sur lequel écoute notre ***service*** est le **`32328`**. Nous savons donc qu’il est possible d’invoquer notre service sur n’importe quelle machine du cluster sur ce port. Listons donc les adresses IPs de nos nœuds :

```sh
dev $ $ kubectl get no -o wide
NAME                    STATUS   ROLES    AGE     VERSION   INTERNAL-IP    EXTERNAL-IP   OS-IMAGE             KERNEL-VERSION   CONTAINER-RUNTIME
k8s-training-dkusr-m1   Ready    master   4h23m   v1.12.1   172.31.20.49   <none>        Ubuntu 16.04.4 LTS   4.4.0-1061-aws   docker://17.3.2
k8s-training-dkusr-n1   Ready    <none>   4h22m   v1.12.1   172.31.20.2    <none>        Ubuntu 16.04.4 LTS   4.4.0-1061-aws   docker://17.3.2
k8s-training-dkusr-n2   Ready    <none>   4h22m   v1.12.1   172.31.22.93   <none>        Ubuntu 16.04.4 LTS   4.4.0-1061-aws   docker://17.3.2
k8s-training-dkusr-n3   Ready    <none>   4h22m   v1.12.1   172.31.24.6    <none>        Ubuntu 16.04.4 LTS   4.4.0-1061-aws   docker://17.3.2
```

La colonne **`INTERNAL-IP`** de nos nœuds est l’information que l’on recherche. Nous pouvons maintenant vérifier que nos pods répondent avec la commande :

```sh
dev $ curl 172.31.20.49:32328
Hello Kubernetes!
```

Vous pouvez vérifier que la commande `cURL` renvoie bien le résultat attendu, quelque soit le nœud du cluster que l’on interroge en remplaçant par une de vos **`INTERNAL-IP`** et votre **`PORT`** associé.

Bravo ! Votre application est désormais disponible dans une **registry privée**, **déployée** dans votre cluster Kubernetes et accessible via un ***service*** !
## 6- Scaling de l’application

L’application kubernetes-application que vous venez de déployer est une application dite stateless. Cela signifie qu’elle ne fait persister ni donnée, ni session. Cette caractéristique facilite grandement la possibilité de mise à l’échelle de cette application. Il s’agit de l’un des [12 factor app](https://12factor.net/fr/).

> **Question**
> Combien de pods ont été déployés dans le chapitre précédent ?

Dans le cas d’une montée en charge, vous aurez besoin d’ajouter des réplicas pour répondre à la demande (spoiler : il est même possible de les ajouter automatiquement mais nous verrons cela plus tard).

> **Exercices**
>- En utilisant la commande `kubectl scale`, passez à 2 réplicas l’application kubernetes-application.
>- Lancez une série de commande `curl` pour valider que votre ***service*** est toujours joignable et fonctionnel.

> **Question**
>
> Quel indice vous montre que le ***service*** redirige vers nos deux pods ?
>
>  _(Describe est votre ami)_

Nous pouvons maintenant faire le test suivant pour prouver la résilience de notre architecture :

```sh
dev $ kubectl delete po --all
pod "kubernetes-app-578bb7d97b-l4nzx" deleted
pod "kubernetes-app-578bb7d97b-x899f" deleted
```
> **Questions**
>- Que s’est-il passé ?
>- Quel est la ressource logique de Kubernetes qui est à l’œuvre ?
  
  ## 2- Configuration du pod grâce à une ConfigMap

Les ***configmaps*** permettent de grouper les configurations consommables par les ***pods***.
Notre objectif va être de faire en sorte que notre application liste le contenu du fichier `/etc/config/message` et l’affiche quand elle est interrogée sur l’URL `/config`.

Nous allons donc ajouter cette configuration à notre conteneur et modifier notre application en conséquence.

`kubectl` permet de générer des ***configmaps*** directement à partir de fichier de configuration. C’est ce que nous allons faire ici.

Commençons par créer le fichier (dans  `~/dalk/deployment/`) :
```sh
dev $ echo -n "Hello from ConfigMap" > message
```
Créons maintenant la ***configmap*** :
```sh
dev $ kubectl create configmap myapp-config --from-file=message
configmap "myapp-config" created
```
> Vérifions que la config map a bien été crée (indice: utiliser la commande describe de kubernetes)

Notre configuration est maintenant disponible. Il faut que nous mettions à jour notre application pour qu’elle réponde lors de l’appel sur l’URL `/config` un fichier.

Ouvrez le fichier `app.py` et modifiez-le afin de faire en sorte que l’application écoute la route `/config` :

```py
# kubernetes-app/app.py
from flask import Flask
from http import HTTPStatus
from werkzeug.exceptions import HTTPException

app = Flask(__name__)

@app.route("/")
def hello():
    return "Hello Kubernetes!\n"

@app.route("/config")                           # <== Ajout
def config():                                   # <== Ajout
    with open('/etc/config/message','r') as f:  # <== Ajout
        return f.read()                         # <== Ajout

@app.errorhandler(HTTPException)
def handle_http_exception(exception: HTTPException):
    return exception.description, exception.code

@app.errorhandler(Exception)
def handle_exception(exception: Exception):
    return HTTPStatus.INTERNAL_SERVER_ERROR.description, HTTPStatus.INTERNAL_SERVER_ERROR
```

Ceci fait, nous allons devoir créer une nouvelle version de notre image docker et la pousser dans la registry. Il est temps de modifier notre `docker-compose.yml` pour préparer ces deux opérations :

```yaml
# kubernetes-app/docker-compose.yml
---
version: '3'
services:
  app:
    build: .
    image: "$REGISTRY_URL/$TRG/app:v0.2" # <== Mise à jour de la version
    ports:
    - "8000:8000"
```

Nous pouvons à présent lancer les commandes de reconstruction et de publication de la nouvelle image :
```sh
dev $ docker compose build
Building app
Sending build context to Docker daemon 22.47 MB
Step 1/10 : FROM python:3.7-alpine
 ---> 74f8760a2a8b
Step 2/10 : LABEL maintainer "<Nom> <Prénom>"
[..]
Successfully built e10b81a9ad54

dev $ docker compose push
Pushing app ($REGISTRY_URL/$TRG/app:v0.2)...
The push refers to a repository [653925650847.dkr.ecr.eu-west-1.amazonaws.com/aug/app]
c0973b845470: Layer already exists
45fcbdaa9e19: Layer already exists
18b267616b22: Layer already exists
67b9b2a215ea: Layer already exists
956940650d5d: Layer already exists
v0.2: digest: sha256:625f435d4b104f163ab68f3d0480299117452fad785400b0374a35e99dbce1a1 size: 2401
```

>>**Notez bien ces étapes** :
>>1. Adaptation de l'application
>>2. Modification du `docker-compose.yml` → pour monter la version l'image
>>3. Lancement de `docker compose build` → pour re-construire l'image
>>4. Lancement de `docker compose up` → pour tester localement l’application
>>5. Lancement de `docker compose push` → pour pousser l'image
>>
>> Vous allez être amenés à les rejouer très régulièrement dans la suite des TPs

Pour rappel, les opérations de _build_ et de _push_, ici ont été effectuées via `docker-compose`, mais si vous l’aviez souhaité, vous auriez pu obtenir les mêmes résultats avec les commandes suivantes :

```sh
dev $ docker image build -t=$REGISTRY_URL/$TRG/app:v0.2 .
dev $ docker image push $REGISTRY_URL/$TRG/app:v0.2
```

Testons maintenant une requête vers notre nouvelle route  (indice: comande curl <IP_UN_DES_NOEUDS>:<NODEPORT>/config...)

Pour l’instant, pas de résultat probant, mais c’est normal. Il est nécessaire de faire plusieurs opérations :
- Préciser qu’à présent nous allons utiliser la version **v0.2** de l’image
- Exposer la ***configmap*** dans notre ***pod***.

> quelles sont les modifications à apporter dans le fichier du ***deployment***, apportez les..

Notons de l’exposition de la ***configmap*** implique 2 nouveaux blocs de configuration :
- Un nouveau Volume qui se base sur la ***configmap*** créée précédemment
- Un point de montage au niveau de notre conteneur qui réutilise le volume initialisé et qui le monte sur le _mountPath_

>Appliquons notre modification :

La nouvelle configuration va automatiquement déployer un nouveau ***pod*** prenant en compte la nouvelle configuration.

> **Exercice**
>
> À l’aide de `kubectl describe pod`, observez la nouvelle configuration.

> Testons maintenant cette nouvelle configuration en faisant de nouveau un curl vers l’ip de notre service (indice: comande curl...)


