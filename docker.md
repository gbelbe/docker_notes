##Image Docker = Classe (qui peut être instanciée autant de fois que l'on veut)
##Container Docker = instance de l'image, qui est executée (donc utilise un process)
##Registry = les images sont stockées dans une registry (publique kitematic == docker store) ou privée.
            gcr.io sur gcp / nexus artifactory ...


##Lister les images téléchargées sur une machine:
    docker image ls

##Telecharger une image d'une registry:
    docker image pull imagename

##Instancier un container a partir d'une image:
    docker container run imagename echo "hello world"

    Instancier un shell a partir d'un container créé a partir d'une image:
    docker container run -i -t busybox  (-i = stdin input /// -t = tty stdout+stderr )
    on peut faire docker container run --help

Instancier un container en mode détaché à partir d'une image:
    docker container run -d  imagename sh -c 'shell script command'

    lister les containers en train de tourner
    docker container ls

    Afficher les logs d'un container: docker container logs -f container_name_ou_id

    Arreter un container:
    docker container stop container_name_ou_id
    docker container ls -a
    docker container rm container_name_ou_id

    supprimer tous les containers arrêtés:
    docker container prune

    idem pour les images:
    docker image ls
    docker image rm image_name
    docker image prune (supprime toutes les images non utilisées par des containers)

En résumé:
  docker client (docker build / docker pull / docker run)
  docker daemon (s'occuppe des instances containers)
  docker registry (héberge les images)

##Docker images layers:
  Les images docker sont écrites en couches.
  On peut utiliser une image comme base d'une précédente, en surchargeant la précédente.

  Si on se base sur une image précédente, pas d'utilisation de disque supplémentaire, ex from:Ubuntu:14.4
  bon par ex pour définir une image de base commune à tous les projets. Comme ça on ne met a jour que cette image de base qui sera reprise par tous les projets.


Note : on peut puller une image, puis travailler sur un container initié à partir de cette imate, y apporter des modifications, puis "commiter" le résultat en créant une nouvelle image.

[on pull une image alpine ]   docker image pull alpine:3.5
[on run un container a partir de l'image]   docker container run -it --name nodeJScontainer alpine3:5
[on installe des binaires]    apk add --update nodejs
                              exit
[on crée une nouvelle image a partir du container]    docker container commit nodeJScontainer mynodeJS (nom de la nouvelle image)

##Partager des images dans dockerHub:
  docker image tag nodeJScontainer  gbelbe/nodejs:1.0
  docker image ls
  docker login
  docker image push gbelbe/nodejs:1.0

Docker commands =>
    $docker <object> <command>
    $docker image pull
    $docker container prune

##Docker et le binding des ports sur la machine host:
    par défaut les services exposés par des containers ne sont pas accessibles, il faut ajouter des bindings de ports de l'interface physique de l'hôte vers les ports du container.
    Association 1:1 entre un port physique et un port d'un container.
    -p 80:8080 (bind le port 80 du host vers le port 8080 du container)

    si l'hote a plusieurs interfaces, on peut choisir depuis quel interface / IP faire le binding. (par défaut, toutes les interfaces (ip) sont bindées)
    ex: pour le ssh, on veut qu'il ne soit visible que de l'intérieur ou en vpn, on le bind sur l'adresse locale.
    -p 192.xx.xx.1:22:22

Ex: couchbase:
docker image pull couchdb:2.1
docker container run --name couchdb1 -d -p 80:5984 couchdb:2.1
- cree un container en mode détaché, appelé couchdb1 et ouvert sur le port 80 sur toutes les interfaces.

##Docker et les volumes
    Note, les documents créés dans des containers sont détruits lorsque le container est détruit.
    Il faut donc creer un volume, et s'occuper de sa persistance pour que les données restent.

    volume =  managés par docker sur le filesystem du host
    bind mount = specifié un mount sur le filesystem du host a un endroit spécifique (non géré par docker en automatique)
    tmpfs = en memoire

Si une image déclare un volume, les données sauvées par le container y seront stockées.

Lister les volumes créés:
$ docker volume ls

ex: on arrête un container puis on supprime le volume créé par le container:
docker container stop couchdb1
docker container rm -v couchdb1
(ou docker volume rm <volume_id>)

on vérifie:
docker volume ls

docker container run --name couchdb1 -d -p 80:5980 -v couchdb_vol:/opt/couchdb/data couchdb:2.1
on créer le container nommé couchdb1, en mode détaché, mappé sur le port 80 au port applicatif 5980, avec le colume nommé couchdb_vol mappé sur le host aux chemin /opt..., le tout a partir de l'image docker couchdb:2.1

Pour lancer un container avec un volume en memoire (tmpfs)

docker container run -ti --tmpfs /test busybox /bin/sh
(lance tout ça a partir d'une image busybox et lance le shell immediatement en mode tty interactif)
