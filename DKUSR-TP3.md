# Docker / Kubernetes - TP3 : Découverte de Kubernetes
> **Objectifs du TP** :
>- Prendre en main l’environnement des TPs
>- Se familiariser avec le client kubectl
>- Découvrir les premiers concepts
>
> **Niveau de difficulté** : Débutant

Pour les questions sufixées par "[AnswerX]", merci de d'inscrire la réponse en remplaçant "<fix me>" dans le fichier "answers/answers_tpX" au niveau de la ligne correspondante '"AnswerX": "<fix me>"'

## 1- Parler avec le Cluster

Nous allons simplement commencer par vérifier que le client `kubectl` fonctionne et parvient à communiquer avec le cluster K8s. Pour ce faire, nous allons simplement lancer la commande :
```sh
dev $ kubectl version --short
```
>**Questions**
>- Quelles sont les versions du client ? [Answer1]
>- Les versions du client et du serveur sont-elles strictement identiques ?


Si nous avons obtenu une réponse sans erreur, nous avons prouvé que :
- Le client est présent et correctement installé
- La configuration du client a permis de se connecter au cluster
- Les accès de l’utilisateurs permettent au moins de demander la version au cluster
- Le cluster est _a minima_ fonctionnel

## 2- Formatage des sorties de kubectl

Une des premières choses à noter avec le client `kubectl`, c’est sa capacité à adapter son format de sortie en fonction de la demande avec l’option `-o` :

```sh
dev $ kubectl version -o yaml
clientVersion:
  buildDate: 2018-09-27T17:05:32Z
  compiler: gc
  gitCommit: 0ed33881dc4355495f623c6f22e7dd0b7632b7c0
  gitTreeState: clean
  gitVersion: v1.12.0
  goVersion: go1.10.4
  major: "1"
  minor: "12"
  platform: linux/amd64
serverVersion:
  buildDate: 2018-09-27T16:55:41Z
  compiler: gc
  gitCommit: 0ed33881dc4355495f623c6f22e7dd0b7632b7c0
  gitTreeState: clean
  gitVersion: v1.12.0
  goVersion: go1.10.4
  major: "1"
  minor: "12"
  platform: linux/amd64
```

Nous allons utiliser cette capacité pour produire une liste formatée des nœuds du cluster Kubernetes.

Pour commencer, voyons la forme standard de sortie de la commande qui permet d’obtenir la liste du premier type de ressources du cluster, les nœuds :

```sh
dev $ kubectl get nodes
NAME                    STATUS   ROLES    AGE     VERSION
k8s-training-dkusr-m1   Ready    master   3h59m   v1.12.1
k8s-training-dkusr-n1   Ready    <none>   3h59m   v1.12.1
k8s-training-dkusr-n2   Ready    <none>   3h59m   v1.12.1
k8s-training-dkusr-n3   Ready    <none>   3h59m   v1.12.1
k8s-training-dkusr-n4   Ready    <none>   3h59m   v1.12.1
```

Cette sortie est simple et peut être adaptée pour fournir plus d’informations. Nous allons essayer à tour de rôles les commandes :
```sh
dev $ kubectl get nodes -o wide
[...]
dev $ kubectl get nodes -o json
[...]
dev $ kubectl get nodes -o yaml
[...]
```

>**Questions**
>- Combien de CPU a le nœud **n1** ? [Answer2]
>- Depuis combien de temps le nœud **n2** est-il enregistré ? [Answer3]
>- Quelle est l’adresse IP interne du nœud **n3** ? [Answer4]

## 3- Formatage avancé des sorties de kubectl

Nous avons vu les formats de sortie fixes, voyons désormais dans quelle mesure il est possible de les personnaliser. Pour cela, d’autres types de sortie que JSON, YAML et `wide` existent.

Les sorties JSON et YAML fournissent directement une structure de données de type `List` qui peut contenir plusieurs éléments, en fonction du nombre de nœuds sur le cluster :
```sh
dev $ kubectl get nodes -o go-template --template='{{ .kind }}{{"\n"}}'
List
dev $ kubectl get nodes -o=template --template='{{ len .items }}{{"\n"}}'
5
```
_A contrario_, si l’on demande une ressource en particulier, son type est directement récupérable :
```sh
dev $ kubectl get no/k8s-training-dkusr-m1 -o go-template --template='{{ .kind }}{{"\n"}}'
Node
```

Il est très simple de récupérer quelques champs spécifiques de la ressource :
```sh
dev $ kubectl get no/k8s-training-dkusr-m1 -o go-template --template='ce {{ .kind }} porte le nom de {{ .metadata.name }}{{"\n"}}'
ce Node porte le nom de k8s-training-dkusr-m1
```

Pour avoir une vue sous forme de tableau personnalisée de tous les éléments, on peut utiliser la sortie **`custom-columns`**, qui reste relativement simple :
```sh
dev $ kubectl get nodes -o=custom-columns=NAME:.metadata.name,RAM:.status.capacity.memory
NAME                    RAM
k8s-training-dkusr-m1   4045068Ki
k8s-training-dkusr-n1   4045068Ki
k8s-training-dkusr-n2   4045068Ki
k8s-training-dkusr-n3   4045068Ki
k8s-training-dkusr-n4   4045068Ki
```

> **Exercice**
>
> En utilisant le format de sortie **`-o=custom-columns`**, produire la liste des nœuds du cluster avec le format suivant :   kubectl get nodes -o=custom-columns=NAME:.metadata.name,ARCH:.status.nodeInfo.architecture,KERNEL:.stat
us.nodeInfo.kernelVersion
> ```
> NAME                    ARCH      KERNEL
> k8s-training-dkusr-m1   amd64     4.4.0-1061-aws
> k8s-training-dkusr-n1   amd64     4.4.0-1061-aws
> k8s-training-dkusr-n2   amd64     4.4.0-1061-aws
> k8s-training-dkusr-n3   amd64     4.4.0-1061-aws
> k8s-training-dkusr-n4   amd64     4.4.0-1061-aws
> ```
