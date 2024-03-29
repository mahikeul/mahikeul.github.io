# Activer le DNS over TLS  (Debian `12` *bookworm*)
- Avant tout, quelques liens vers une liste de serveurs DNS respectueux :
  - https://www.opennic.org/
  - https://www.fdn.fr/actions/dns/
- Passer la gestion du `/etc/resolv.conf` à `systemd` plutôt que le laisser à la charge de `NetworkManager` :
```shell
sudo apt install systemd-resolved
```
- Activer le service :
```shell
sudo systemctl enable systemd-resolved
```
- Le fichier `/etc/resolv.conf` devient alors un lien symbolique (vers `/run/systemd/resolve/stub-resolv.conf`), détecté par `NetworkManager` qui passera alors le traitement des serveurs DNS à `systemd-resolved` (expliqué dans la section `dns` de `man NetworkManager.conf`).  
- Pour afficher la configuration de résolution de nom, au lieu de consulter le fichier `/etc/resolv.conf`, maintenant utiliser la commande :
```shell
resolvectl status
```
- Lister les connexions gérées par `NetworkManager` pour identifier celles sur lesquelles activer le *DNS over TLS* :
```shell
nmcli connection show
```
- Afficher le paramétrage d'une connexion, en remplaçant `<UUID>` par celui retourné dans la commande précédente :
```shell
nmcli connection show <UUID>
```
- Pour ajouter (en préfixant l'option par "`+`") ou supprimer (en préfixant l'option par "`-`") des serveurs DNS associés à une connexion (option `ipv4.dns` pour le protocole IPv4, et `ipv6.dns` pour le protocole IPv6) :
```shell
nmcli connection modify <UUID> +ipv4.dns 80.67.169.12 +ipv4.dns 80.67.169.40
```
- Pour chaque connexion ciblée, activer l'option `connection.dns-over-tls` (`-1`: par défaut, `0`: désactivé, `1`: [opportuniste](https://gitlab.freedesktop.org/NetworkManager/NetworkManager/-/issues/258#note_444510), `2`: activé) :
```shell
nmcli connection modify <UUID> connection.dns-over-tls 2
```
- Redémarrer le service `NetworkManager` pour prise en compte :
```shell
sudo systemctl restart NetworkManager.service
```
## Remarque avec `podman` (et _certainement_ `docker`)
`podman`, par mesure de sécurité, n'interroge pas les adresses locales lorsque le réseau utilisé est celui par défaut : les conteneurs ne pourront pas interroger le serveur DNS local (en `127.0.0.53`) mis à disposition par `systemd-resolved` ([&#x21aa; source](https://github.com/containers/podman/issues/3277)).  
- La solution est de créer un nouveau réseau ([&#x21aa; source](https://blog.podman.io/2023/02/the-container-name-resolution-conundrum/)) :
```shell
podman network create {nouveau réseau}
```
- Ou d'utiliser un réseau autre que celui par défaut (`podman`).  
Par exemple, utiliser le réseau `kind` créé lors de la [mise en place d'un environnement Kubernetes local](../k8s-local/README.md).  
Pour voir la liste des réseaux créés :
```shell
podman network ls
```  
- puis de :
  - soit paramétrer le réseau au lancement du conteneur, en ajoutant l'option `--network {nouveau réseau}`.
  - soit configuer ce nouveau réseau comme réseau par défaut, en ajoutant la ligne `default_network = "{nouveau réseau}"` à la section `[network]` du fichier `/etc/containers/containers.conf`.