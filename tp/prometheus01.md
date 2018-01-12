# Prometheus

https://docs.docker.com/engine/admin/prometheus/#configure-and-run-prometheus


--------------



* cAdvisor va exposer un endpoint http://cadvisor:8080/metrics avec l’ensemble des metrics des containers au moment t.

(NB dans les version récente on peut directement configurer docker en endpoint pour prometheus, cf lien précédent.)

* Prometheus va requêter toute les x secondes l’endpoint de cAdvisor et stocker les metrics dans sa base de données.

* Grafana va afficher les metrics de Prometheus sous forme de graphs.

```
## On démarre des container qui génère de l'activitée

docker run -d --cpuset-cpus 0 --cpu-shares 256 stress
docker run -d ubuntu sleep infinity
docker run -d alpine ping 8.8.8.8

Dans un docker-compose.yml :

```
