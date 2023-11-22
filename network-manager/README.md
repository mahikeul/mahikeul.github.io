# NetworkManager
## Quelques commandes utiles
- Pour qu'une connexion (_VPN_ par exemple) ne soit jamais la _route_ par défaut (pour pouvoir accéder à Internet si la connexion cible un réseau fermé) :
```shell
nmcli connection modify <UUID> ipv4.never-default true
```
- Pour avoir des informations sur un paramètre d'une connexion, passer en mode édition avec la commande suivante sur la connexion `<UUID>` visée, puis invoquer `describe {nom-du-paramètre}` (par exemple, `describe ipv4.never-default`) :
```shell
nmcli connection edit <UUID>
```
