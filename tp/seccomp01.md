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


Dans ce cas les syscall ne sont plus limité on peut par exemple créer un namespace avec `unshare` à l'interieur d'un container. NB : on peut utiliser strace pour suivre les appels systems (Ex. `strace -c -f -S name ls 2>&1 1>/dev/null`)



## syscall par défaut mais sans chmod autorisé

Le profile à utiliser
https://raw.githubusercontent.com/docker/labs/master/security/seccomp/seccomp-profiles/default-no-chmod.json

```
docker run --rm -it --security-opt seccomp=./seccomp-profiles/default-no-chmod.json alpine sh

chmod 777 / -v
chmod: /: Operation not permitted
```
