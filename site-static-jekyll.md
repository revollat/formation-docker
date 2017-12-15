# But

* 1er service docker : Jekyll qui va nous permettre de générer un site web statique.
* 2eme service docker : Serveur web nginx pour servir le site statique.

## Service 1 : Jekyll

```bash
$ mkdir jekyll
$ cd jekyll
$ vi Dockerfile
```

```Dockerfile
FROM ubuntu:16.04
MAINTAINER James Turnbull <james@example.com>
ENV REFRESHED_AT 2016-06-01

RUN apt-get -qq update
RUN apt-get -qq install ruby ruby-dev build-essential nodejs
RUN gem install --no-rdoc --no-ri jekyll -v 2.5.3

VOLUME /data
VOLUME /var/www/html
WORKDIR /data

ENTRYPOINT [ "jekyll", "build", "--destination=/var/www/html" ]
```

```bash
docker build -t jekyll .
```
## Service 2 : Apache
```Dockerfile
FROM ubuntu:16.04
MAINTAINER James Turnbull <james@example.com>

RUN apt-get -qq update
RUN apt-get -qq install apache2

VOLUME [ "/var/www/html" ]
WORKDIR /var/www/html

ENV APACHE_RUN_USER www-data
ENV APACHE_RUN_GROUP www-data
ENV APACHE_LOG_DIR /var/log/apache2
ENV APACHE_PID_FILE /var/run/apache2.pid
ENV APACHE_RUN_DIR /var/run/apache2
ENV APACHE_LOCK_DIR /var/lock/apache2

RUN mkdir -p $APACHE_RUN_DIR $APACHE_LOCK_DIR $APACHE_LOG_DIR

EXPOSE 80

ENTRYPOINT [ "/usr/sbin/apache2" ]
CMD ["-D", "FOREGROUND"]
```

```bash
docker build -t apache .
```

## Site statique de démo

```bash
$ git clone https://github.com/turnbullpress/james_blog.git
$ docker run -v $PWD/james_blog:/data/ --name james_blog jekyll
$ docker run -d -P --volumes-from james_blog apache
```

### Modif du site

modifier le titre dans `_config.yml`

```
$ docker start james_blog
james_blog

$ docker logs james_blog                                                                                                                        ⏎
Configuration file: /data/_config.yml
            Source: /data
       Destination: /var/www/html
      Generating...
                    done.
 Auto-regeneration: disabled. Use --watch to enable.
Configuration file: /data/_config.yml
            Source: /data
       Destination: /var/www/html
      Generating...
                    done.
 Auto-regeneration: disabled. Use --watch to enable.
```

### Sauvegarde du site statique

```
$ docker run --rm --volumes-from james_blog -v $(pwd):/backup ubuntu tar cvf /backup/james_blog_backup.tar /var/www/html
```
