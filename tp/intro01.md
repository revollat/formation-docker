Ressources :
https://github.com/docker/labs/blob/master/beginner/chapters/alpine.md
http://training.play-with-docker.com/beginner-linux/
https://docs.docker.com/get-started/part2/#dockerfile

# Votre premier container

`docker run alpine ls -l`

* le client docker contact le daemon
* le deamon vérifie si l'image est présente en local sinon il la télécharge depuis le Hub.
* Le daemon créer un container et lance une commande à l'interieur
* le daemon stream la sortie du processus vers le client docker


autre test

`docker run alpine echo "hello from alpine"`

### lister les containers

En cours d'éxecution :
`docker container ls`

pour voir aussi les containers arretés :
`docker container ls  --all`

Formattage de la sortie

`docker container ls --no-trunc --format "{{.ID}}\t{{.Command}}"`

https://docs.docker.com/engine/reference/commandline/ps/#formatting

### S'attacher au STDIN/STDOUT du processus pour interagir

`docker run -it alpine /bin/sh`


* -t means "allocate a terminal."
* -i means "connect stdin to the terminal."


Se détacher avec ^P^Q

S'attacher au dernier (option -l) container lancé :

`docker attach $(docker container ls -lq)`

### Tourner en mode tache de fond

```
$ docker run jpetazzo/clock
Thu Jan 11 09:43:26 UTC 2018
Thu Jan 11 09:43:27 UTC 2018
Thu Jan 11 09:43:28 UTC 2018
Thu Jan 11 09:43:29 UTC 2018
Thu Jan 11 09:43:30 UTC 2018
```
Lancer en tache de fond

```
$ docker run -d --name test-clock jpetazzo/clock
```

On voit le pid du processus qui tourne sur la machine hote.
```
$ docker container top test-clock
```

Voir les logs en live

```
$ docker container logs -f --tail 1 test-clock
```

En démarrer plusieurs

```
$ docker run -d jpetazzo/clock
$ docker run -d jpetazzo/clock
$ docker run -d jpetazzo/clock
...
```

## Modif à chaud d'un container

```
$ docker run -it ubuntu
root@bec3b4a0948f:/# figlet Hello
bash: figlet: command not found
root@bec3b4a0948f:/# apt-get update
root@bec3b4a0948f:/# apt-get install figlet
root@bec3b4a0948f:/# figlet Hello
 _   _      _ _
| | | | ___| | | ___
| |_| |/ _ \ | |/ _ \
|  _  |  __/ | | (_) |
|_| |_|\___|_|_|\___/

root@bec3b4a0948f:/#
```

On peut arreter le container et le redémarrer, figlet est toujours présent par contre si on le supprimer et qu'on redémarrer à partir de la meme image on à perdu figlet.
Pour le conserver, il faut commit le container avant de le supprimer.

## Lancer un container MySQL

```
$ docker container run \
--detach \
--name mydb \
-e MYSQL_ROOT_PASSWORD=passwd \
mysql:latest

$ docker container ls

$ docker container logs mydb

$ docker container top mydb # INFOS SUR LE PROCESS LANCE

$ docker exec -it mydb \
mysql --user=root --password=passwd --version

$ docker exec -it mydb sh
```

## Dockerfile intro

On va créer un dockerfile pour notre outil figlet

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN apt-get install figlet
```

On build l'image.
Attention : les commandes lancées par les instructions RUN ne doivent pas etres interactives, par example on passera en général l'option -y à apt-get install. i.e réponds y/yes à toutes les éventuels questions en auto.

`docker image build --tag ubuntu-figlet .`

#### Explications de ce qui se passe :

* Sending the build context to Docker

C'est le dossier indiqué en paramètre ici "." qui est passé (sous forme d'archive au demon docker). Cela permet d'utiliser un demon docker distant pour builder à partir de fichiers locaux.
(Attention à ce que le dossier contexte ne soit pas trop gros) On peut aussi utiliser un .dockerignore (un peu comme un .gitignore)

* Step by Step

On part de l'image de base (2d696327ab2e) et on créer un nouveau layers pour chaque instructions du dockerfile (Ex. 46d42b95f209), ce nouveau layer sert de base pour l'instruction suivante.

**NB** : des layers intermédiaires peuvent etre créer et supprimés Ex. Removing intermediate container 077bc0a614e5

```
$ docker image build --tag ubuntu-figlet .
Sending build context to Docker daemon  5.157MB                                                     
Step 1/3 : FROM ubuntu           
 ---> 2d696327ab2e
Step 2/3 : RUN apt-get update
 ---> Running in 077bc0a614e5
Get:1 http://archive.ubuntu.com/ubuntu xenial InRelease [247 kB]
Get:2 http://security.ubuntu.com/ubuntu xenial-security InRelease ...
...
Reading package lists...
 ---> 46d42b95f209
Removing intermediate container 077bc0a614e5
Step 3/3 : RUN apt-get install figlet
 ---> Running in 405047fa6352
...
...
```

* Cache

Si on rebuild le cache est utilisé car les instruction sont les memes. Sinon faire `docker build --no-cache`

#### Lancer l'Image

```
$ docker run ubuntu-figlet
# figlet hello
_          _ _       
| |__   ___| | | ___  
| '_ \ / _ \ | |/ _ \
| | | |  __/ | | (_) |
|_| |_|\___|_|_|\___/
```

#### CMD et Entrypoint

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN ["apt-get", "install", "figlet"]
CMD figlet -f script hello
```

CMD permet de définir la commande par défaut.
Si il y en a plusieurs dans le dockerfile, seul le dernier est pros en compte.

`docker run ubuntu-figlet`

On peut surcharger CMD au lancement du container

`docker run -it ubuntu-figlet bash`

Maintenant on définit un entrypoint

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN ["apt-get", "install", "figlet"]
ENTRYPOINT ["figlet", "-f", "script"]
```

```
$ docker run -it ubuntu-figlet Docker
  ____             _
 (|   \           | |
  |    | __   __  | |   _   ,_
 _|    |/  \_/    |/_) |/  /  |
(/\___/ \__/ \___/| \_/|__/   |_/
```

CMD et Entrypoint

```Dockerfile
FROM ubuntu
RUN apt-get update
RUN ["apt-get", "install", "figlet"]
ENTRYPOINT ["figlet", "-f", "script"]
CMD ["hello world"]
```

```
$ docker run -it ubuntu-figlet
 _          _   _                             _
| |        | | | |                           | |    |
| |     _  | | | |  __             __   ,_   | |  __|
|/ \   |/  |/  |/  /  \_  |  |  |_/  \_/  |  |/  /  |
|   |_/|__/|__/|__/\__/    \/ \/  \__/    |_/|__/\_/|_/

$ docker run -it ubuntu-figlet docker
                 _
   |            | |
 __|   __   __  | |   _   ,_
/  |  /  \_/    |/_) |/  /  |
\_/|_/\__/ \___/| \_/|__/   |_/
```

Override de l'entry point
```
$ docker run -it --entrypoint bash ubuntu-figlet
```

## Image d'un simple site web

clone https://github.com/dockersamples/linux_tweet_app.git

```
$ docker image build --tag linux_tweet_app:1.0 .

$ docker container run \
 --detach \
 --publish 8088:80 \
 --name linux_tweet_app \
 linux_tweet_app:1.0

$ docker container rm --force linux_tweet_app
```

### Bind mount source site pour travailler en live dessus

```
docker container run \
--detach \
--publish 8088:80 \
--name linux_tweet_app \
--mount type=bind,source="$(pwd)",target=/usr/share/nginx/html \
linux_tweet_app:1.0
```
faire une modif Ex. `cp index-new.html index.html`


Arreter le container  `docker rm --force linux_tweet_app`

Rebuilder l'image avec un nouveau tag 2.0

`docker image build --tag linux_tweet_app:2.0 .`

Lancer la nouvelle version (l'image de l'ancienne version est toujours utilisable, i.e. version 1.0)
```
docker container run \
 --detach \
 --publish 8088:80 \
 --name linux_tweet_app \
 linux_tweet_app:2.0
```
### push dans le hub

```
docker login
$ docker image tag linux_tweet_app:1.0 revollat/linux_tweet_app:1.0  
$ docker image tag linux_tweet_app:2.0 revollat/linux_tweet_app:2.0
$ docker image push revollat/linux_tweet_app:1.0
$ docker image push revollat/linux_tweet_app:2.0
```

NOTEZ LES LAYERS DEJA EXISTANTS QUAND ON POUSSE LA V2
```
671458c4618a: Pushed
74f51e7dd1ab: Pushed
110566462efa: Layer already exists
305e2b6ef454: Layer already exists
24e065a5f328: Layer already exists
```


## Une application python

Dockerfile

```Dockerfile
# Use an official Python runtime as a parent image
FROM python:2.7-slim

# Set the working directory to /app
WORKDIR /app

# Copy the current directory contents into the container at /app
ADD . /app

# Install any needed packages specified in requirements.txt
RUN pip install --trusted-host pypi.python.org -r requirements.txt

# Make port 80 available to the world outside this container
EXPOSE 80

# Define environment variable
ENV NAME World

# Run app.py when the container launches
CMD ["python", "app.py"]
```


requirements.txt
```
Flask
Redis
```

app.py
```python
from flask import Flask
from redis import Redis, RedisError
import os
import socket

# Connect to Redis
redis = Redis(host="redis", db=0, socket_connect_timeout=2, socket_timeout=2)

app = Flask(__name__)

@app.route("/")
def hello():
    try:
        visits = redis.incr("counter")
    except RedisError:
        visits = "<i>cannot connect to Redis, counter disabled</i>"

    html = "<h3>Hello {name}!</h3>" \
           "<b>Hostname:</b> {hostname}<br/>" \
           "<b>Visits:</b> {visits}"
    return html.format(name=os.getenv("NAME", "world"), hostname=socket.gethostname(), visits=visits)

if __name__ == "__main__":
    app.run(host='0.0.0.0', port=80)
```
### build
```
docker build -t pythonhello .
docker run -p 4000:80 pythonhello
```
