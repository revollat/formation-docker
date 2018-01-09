# Capabilities


Docker démarre les containers avec un ensmble restreint de capabilities i.e. une whitelist (cf. https://github.com/moby/moby/blob/master/oci/defaults.go#L14-L30)



Par defaut on possède la capability "CAP_CHOWN", par exemple, on peut changer le propriétaire de `/`

 `docker run --rm -it alpine chown nobody /`

On peut aussi supprimer toutes les cap sauf cap_chown

 `docker run --rm -it --cap-drop ALL --cap-add CHOWN alpine chown nobody /`


Si on démarre le container sans cap_chown :

 ```
 docker run --rm -it --cap-drop CHOWN alpine chown nobody /
 chown: /: Operation not permitted
 echo $?
 ```

Docker ne permet pas encore d'ajouter des capabilities à un utilisateur non-root (on lance le container avec `-u nobody`) :

```
 docker run --rm -it --cap-add chown -u nobody alpine chown nobody /
 chown: /: Operation not permitted
```
