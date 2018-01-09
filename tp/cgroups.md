# Cgroups

`sudo apt-get install htop`

Dockerfile du "stresser"

```dockerfile
FROM ubuntu:latest
RUN apt-get update && apt-get install -y stress
CMD stress -c 2
```

```
docker build -t stress
docker run -d --name stresser stress
```

htop montre que le CPU est à  100% le load average grimpe ...

![](https://camo.githubusercontent.com/011bb47d3a0f23ee304647fd0464120134a48271/687474703a2f2f692e696d6775722e636f6d2f4c4232794e30742e706e67)


# Pour que le container utilise que le premier core

`docker run -d --name stresser --cpuset-cpus 0 stress`

Ls 2 process stress utilisent chacun 50% du core 0 de la machine :

![](https://camo.githubusercontent.com/a56c427a7063beaba1d2ef021b854cf99851bda3/687474703a2f2f692e696d6775722e636f6d2f494a50333162502e706e67)

## Pareil mais Container-1 sera limité à 75% du core 0 et container-2 25%

```
docker run -d --name container-2 --cpuset-cpus 0 --cpu-shares 256 stress
docker run -d --name container-1 --cpuset-cpus 0 --cpu-shares 768 stress    
```

# Limiter une fork bomb avec `--pids-limit`

`docker run --rm -it --pids-limit 200 debian:jessie bash`

puis

```
# :(){ :|: & };:
...
...
bash: fork: Resource temporarily unavailable
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable
bash: fork: Resource temporarily unavailable
bash: fork: Resource temporarily unavailable
bash: fork: Resource temporarily unavailable
bash: fork: Resource temporarily unavailable
bash: fork: retry: No child processes
bash: fork: Resource temporarily unavailable
...
...
```
