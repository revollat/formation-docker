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

### autre test
`docker run alpine echo "hello from alpine"`

### lister les containers

En cours d'éxecution :
`docker container ls`

pour voir aussi les containers arretés :
`docker container ls  --all`

### S'attacher au STDIN/STDOUT du processus pour interagir

`docker run -it alpine /bin/sh`

### Tourner en mode tache de fond

```
$ docker run -d alpine sleep 1234
b99e287a7d0be321c97bd0560f270466267266bd6acb027284aa5a5a61b6b65e

$ ps aux | grep 1234
root     25965  0.1  0.0   1508     4 ?        Ss   15:17   0:00 sleep 1234
```

On voit le pid du processus qui tourne sur la machine hote.

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
