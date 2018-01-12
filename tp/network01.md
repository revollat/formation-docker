# Network

## L'interface bridge de la machine hote

`docker network ls`

```bash
$ sudo apt-get install bridge-utils

$ brctl show
bridge name     bridge id               STP enabled     interfaces
...
...
docker0         8000.02422e4da784       no

$ ip addr show docker0
3: docker0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default
   link/ether 02:42:2e:4d:a7:84 brd ff:ff:ff:ff:ff:ff
   inet 172.17.0.1/16 scope global docker0
      valid_lft forever preferred_lft forever
   inet6 fe80::42:2eff:fe4d:a784/64 scope link
      valid_lft forever preferred_lft forever

$ docker run -dt ubuntu sleep infinity
$ docker run -dt ubuntu sleep infinity

# Notez les l'interfaces veth des containers connectés
$ brctl show
bridge name     bridge id               STP enabled     interfaces
docker0         8000.02422e4da784       no              veth1fd9c18

                                                        veth4152a4c

```

`docker network inspect bridge`

On voit les 2 containers attachés :

```json
"Containers": {
    "05fff56d2e879656e121a23b0196462e715dc1854126bd6cea8bcdf7503e2313": {
        "Name": "laughing_leavitt",
        "EndpointID": "687ecc196152ff301cb36bd82eecbe84af13e8a873fb4a303118e401e56fc3b8",
        "MacAddress": "02:42:ac:11:00:03",
        "IPv4Address": "172.17.0.3/16",
        "IPv6Address": ""
    },
    "7bdfdd524ea453384215d0b476713d718015c2ca10247229f88f1be29a4fb554": {
        "Name": "epic_meninsky",
        "EndpointID": "5c0561acbaa96093b243cc2be7716f8de7d201c40f93f838cbee58cdfdbad654",
        "MacAddress": "02:42:ac:11:00:02",
        "IPv4Address": "172.17.0.2/16",
        "IPv6Address": ""
    }
},
```

`docker network create dev`

`docker network ls`

`docker run -d --name es --net dev elasticsearch:2`

Ping depuis un autre container sur le meme réseau

```
$ docker run -ti --net dev alpine sh
/ # ping es
PING es (172.18.0.2): 56 data bytes
64 bytes from 172.18.0.2: seq=0 ttl=64 time=0.196 ms
64 bytes from 172.18.0.2: seq=1 ttl=64 time=0.156 ms
^C
--- es ping statistics ---
2 packets transmitted, 2 packets received, 0% packet loss
round-trip min/avg/max = 0.156/0.176/0.196 ms
/ #
```

ça ne fontionne que parce qu'on est sur le meme réseau.

Grace aux network alias on peut avoir le meme alias pour plusieurs instance d'un container.

```
$ docker network create prod
a75553db716e6f1c858fb3007de1897e577abae351febfeb39f33aec4fe20368
$ docker run -d --name prod-es-1 --net-alias es --net prod elasticsearch:2
7b7acbdfd88bab63d80597e69455870ce22628e2b7fc03706bcdb17638fe9d7e
$ docker run -d --name prod-es-2 --net-alias es --net prod elasticsearch:2
3858f29ca6fa170eb1934376ed9a88447b40bbfea0d0e9386fcd82baab700a2f                                    

$ docker run --net prod --rm alpine nslookup es

nslookup: can't resolve '(null)': Name does not resolve
Name:      es
Address 1: 172.19.0.2 prod-es-1.prod
Address 2: 172.19.0.3 prod-es-2.prod
```

Si on fait un curl sur l'alias on tape alternativement sur chacun des containers (DNS round robin)

```
$ docker run --rm --net prod appropriate/curl -s es:9200
{
  "name" : "Sam Sawyer",
...
}
$ docker run --rm --net prod appropriate/curl -s es:9200
{
  "name" : "Ethan Edwards",
...
}
```
