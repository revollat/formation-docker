# Layers images
## Réutilisation des layers dans les images

Dockerfile.base

```dockerfile
FROM alpine
RUN touch /test
```

Dockerfile

```dockerfile
FROM alpine
RUN touch /test
RUN touch /autre-fichier
```

```
docker build -t revollat/layers1 -f Dockerfile.base .
docker build -t revollat/layers2 -f Dockerfile .
```

L'image Layers2 réutilise les layers déja crées par le build de l'image layers1.
Seule la nouvelle couche ff483e925877 (i.e. RUN touch /autre-fichier) est ajoutée.

```
ubuntu@vps319836 ~/t/tmp ❯❯❯ docker image history revollat/layers1  && docker image history revollat/layers2
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ca3c65e5b8e7        9 minutes ago       /bin/sh -c touch /test                          0B
e21c333399e0        5 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           5 weeks ago         /bin/sh -c #(nop) ADD file:2b00f26f6004576...   4.14MB
IMAGE               CREATED             CREATED BY                                      SIZE                COMMENT
ff483e925877        8 minutes ago       /bin/sh -c touch /autre-fichier                 0B
ca3c65e5b8e7        9 minutes ago       /bin/sh -c touch /test                          0B
e21c333399e0        5 weeks ago         /bin/sh -c #(nop)  CMD ["/bin/sh"]              0B
<missing>           5 weeks ago         /bin/sh -c #(nop) ADD file:2b00f26f6004576...   4.14MB
ubuntu@vps319836 ~/t/tmp ❯❯❯
```

## Modifier et commit des modifs

```
$ docker run -it revollat/layers2
# echo "Ceci n'est pas un test" > /autre-fichier
```

Se détacher avec Ctrl + P + Q
On peut voir les différences entre l'image d'origine et de container avec

```
$ docker container diff $(docker container ls -lq)
C /autre-fichier
C /root
A /root/.ash_history
```

Puis commit du dernier container lancé et on l'appel revollat/layers3

```
$ docker container commit $(docker container ls -lq) revollat/layers3
```

Diff des images layers2 et layers3 :

```
docker image history revollat/layers2  && docker image history revollat/layers3
```

## Docker hub, Pull, Push, Tags

Examples avec pull, push, tags (expliquer avec l'exmple des tags ngnix : latest, alpine, ubuntu, versions, ... cf .https://hub.docker.com/_/nginx/)

## Ou se trouve ces layers ?
```
# docker image inspect revollat/layers1 | jq
```
