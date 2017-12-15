# Stack multicontainer

* Un container avec une app Node.js liéé à :
  * sdz
  * Un redis pour l'etat liéé à 2 autres container redis pour créer un cluster distribué des données
  * un container pour capturer les logs applicatif


Récupérer les sources depuis : https://github.com/turnbullpress/dockerbook-code.git

et allez dans le dossier `cd code/6/node`


Cf chapitre 6 du livre 6.3

```
docker run -d -h redis_primary --net express --name redis_primary redisprimary

docker run -ti --rm --volumes-from redis_primary ubuntu cat /var/log/redis/redis-server.log

docker run -d -h redis_replica1 --name redis_replica1 --net express redisreplica

docker run -ti --rm --volumes-from redis_replica1 ubuntu cat /var/log/redis/redis-replica.log

docker run -d -h redis_replica2 --name redis_replica2 --net express redisreplica

docker run -d --name nodeapp -p 3000:3000 --net express nodejsapp

docker run -d --name logstash --volumes-from redis_primary --volumes-from nodeapp logstash
```

accéder à http://localhost:3000/hello/oliv

```bash
docker logs logstash

$ docker exec -it redis_primary redis-cli                                                                              master ✱
127.0.0.1:6379> keys *
1) "sess:sK-Cn_PclEoTkXqBHty6bJw9"
127.0.0.1:6379> mget sess:sK-Cn_PclEoTkXqBHty6bJw9
1) "{\"cookie\":{\"originalMaxAge\":2592000000,\"expires\":\"2018-01-14T10:06:27.523Z\",\"httpOnly\":true,\"path\":\"/\"}}"
127.0.0.1:6379>

```
