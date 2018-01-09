# Seccomp

Pour utiliser le profile seccomp deny.json suivant :

```json
{
	"defaultAction": "SCMP_ACT_ERRNO",
	"architectures": [
		"SCMP_ARCH_X86_64",
		"SCMP_ARCH_X86",
		"SCMP_ARCH_X32"
	],
	"syscalls": [
	]
}
```

Action |	Description
--- | ---
SCMP_ACT_KILL |	Kill with a exit status of 0x80 + 31 (SIGSYS) = 159
SCMP_ACT_TRAP |	Send a SIGSYS signal without executing the system call
SCMP_ACT_ERRNO |	Set errno without executing the system call
SCMP_ACT_TRACE |	Invoke a ptracer to make a decision or set errno to -ENOSYS
SCMP_ACT_ALLOW |	Allow

** Aucun syscall ne sera autorisé **

on lance le container en désactivant apparmor et en ajoutant toutes les capabilities pour voir l'effet seul des profils seccomp

`docker run --rm -it --cap-add ALL --security-opt apparmor=unconfined --security-opt seccomp=seccomp-profiles/deny.json alpine sh`

Il y a une erreur car docker n'a pas assez de droit sur les syscall ne serait-ce que pour se lancer le container...


Par defaut si aucun profile seccomp est spécifié c'est celui par defaut qui st utilisé :
https://github.com/moby/moby/blob/master/profiles/seccomp/default.json

## Pour lancer un container **sans** profile seccomp :

`docker run --rm -it --security-opt seccomp=unconfined debian:jessie sh`


Dans ce cas les syscall ne sont plus limité on peut par exemple créer un namespace avec `unshare` à l'interieur d'un container.



## syscall par défaut mais sans chmod autorisé

Le profile à utiliser
https://raw.githubusercontent.com/docker/labs/master/security/seccomp/seccomp-profiles/default-no-chmod.json

```
docker run --rm -it --security-opt seccomp=./seccomp-profiles/default-no-chmod.json alpine sh

chmod 777 / -v
chmod: /: Operation not permitted
```
# Strace des containers

on peut utiliser strace pour suivre les appels systems, mais dans un container qui n'a pas de shell ni l'outil strace ?

On lance un ngninx :

`docker run -d -P --name web nginx:alpine`

On lance un deuxième container alpine avec strace installé et la commande executée est `CMD ["strace", "-f", "-p", "1"]`

`docker run -it --pid=container:web  --cap-add sys_admin --cap-add sys_ptrace sjourdan/strace`


On peut voir les syscall suite à une requete :

```
...
[{EPOLLIN, {u32=1995264032, u64=140219792576544}}], 512, -1, NULL, 8) = 1
[pid    14] accept4(6, {sa_family=AF_INET, sin_port=htons(36446), sin_addr=inet_addr("172.17.0.1")}, [112->16], SOCK_NONBLOCK) = 4
[pid    14] epoll_ctl(10, EPOLL_CTL_ADD, 4, {EPOLLIN|EPOLLRDHUP|EPOLLET, {u32=1995264512, u64=140219792577024}}) = 0
[pid    14] epoll_pwait(10, [{EPOLLIN, {u32=1995264512, u64=140219792577024}}], 512, 60000, NULL, 8) = 1
[pid    14] recvfrom(4, "GET /?ferf HTTP/1.1\r\nHost: 127.0"..., 1024, 0, NULL, NULL) = 457
[pid    14] stat("/usr/share/nginx/html/index.html", {st_mode=S_IFREG|0644, st_size=612, ...}) = 0
[pid    14] open("/usr/share/nginx/html/index.html", O_RDONLY|O_NONBLOCK) = 5
[pid    14] fstat(5, {st_mode=S_IFREG|0644, st_size=612, ...}) = 0
[pid    14] writev(4, [{iov_base="HTTP/1.1 304 Not Modified\r\nServe"..., iov_len=180}], 1) = 180
[pid    14] write(7, "172.17.0.1 - - [09/Jan/2018:15:1"..., 160) = 160
[pid    14] close(5)                    = 0
[pid    14] setsockopt(4, SOL_TCP, TCP_NODELAY, [1], 4) = 0
[pid    14] epoll_pwait(10,
...
```

NB : on peut aussi envoyer un signal HUP (hangup) à nginx pour reloader à chaud le config et observer le resultat des appels system :

depuis un autre terminal :
`docker kill -s HUP web`

On observe :

```
...
[pid     1] open("/etc/nginx/nginx.conf", O_RDONLY) = 8
...
```
