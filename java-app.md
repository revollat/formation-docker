# Java application server

Pipeline docker en 2 étapes :

* Image qui va récupérer un WAR depuis une URL et le stocker dans un volume
* Image avec un Tomcat qui va faire tourner les WAR

## Fetcher

```Dockerfile
FROM ubuntu:16.04
MAINTAINER James Turnbull <james@example.com>
ENV REFRESHED_AT 2016-06-01

RUN apt-get -qq update
RUN apt-get -qq install wget

VOLUME [ "/var/lib/tomcat7/webapps/" ]
WORKDIR /var/lib/tomcat7/webapps/

ENTRYPOINT [ "wget" ]
CMD [ "--help" ]
```

```
docker build -t fetcher .
```

```
docker run -t -i --name sample fetcher https://tomcat.apache.org/tomcat-7.0-doc/appdev/sample/sample.war
```

Le ficheir téélchargé se trouve dnas :

```bash
$ docker inspect -f "{{ range .Mounts }}{{.}}{{end}}" sample

{volume bc2e76766fe9c709784d62fda1991d023633d2b507f95c1ea5b44c6fee8d6160 /var/lib/docker/volumes/bc2e76766fe9c709784d62fda1991d023633d2b507f95c1ea5b44c6fee8d6160/_data /var/lib/tomcat7/webapps local  true }
```

## tomcat

```Dockerfile
FROM ubuntu:16.04
MAINTAINER James Turnbull <james@example.com>
ENV REFRESHED_AT 2016-06-01

RUN apt-get -qq update
RUN apt-get -qq install tomcat7 default-jdk

ENV CATALINA_HOME /usr/share/tomcat7
ENV CATALINA_BASE /var/lib/tomcat7
ENV CATALINA_PID /var/run/tomcat7.pid
ENV CATALINA_SH /usr/share/tomcat7/bin/catalina.sh
ENV CATALINA_TMPDIR /tmp/tomcat7-tomcat7-tmp

RUN mkdir -p $CATALINA_TMPDIR

VOLUME [ "/var/lib/tomcat7/webapps/" ]

EXPOSE 8080

ENTRYPOINT [ "/usr/share/tomcat7/bin/catalina.sh", "run" ]

```

```
$ docker build -t tomcat .
$ docker run --name sample_app --volumes-from sample -d -P tomcat
$ docker port sample_app 8080
```

Allez sur l'URL en ajoutant **/sample/** Ex. http://localhost:32769/sample/
