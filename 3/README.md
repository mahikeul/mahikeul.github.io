***Activer le DNS over TLS  (Debian `12` *bookworm*)***
---
<br />

- Avant tout, quelques liens vers une liste de serveurs DNS respectueux :
  - https://www.opennic.org/
  - https://www.fdn.fr/actions/dns/
- Passer la gestion du `/etc/resolv.conf` à `systemd` plutôt que le laisser à la charge de `NetworkManager` :
```
sudo apt install systemd-resolved
```
- Activer le service :
```
sudo systemctl enable systemd-resolved
```
- Le fichier `/etc/resolv.conf` devient alors un lien symbolique (vers `/run/systemd/resolve/stub-resolv.conf`), détecté par `NetworkManager` qui passera alors le traitement des serveurs DNS à `systemd-resolved` (expliqué dans la section `dns` de `man NetworkManager.conf`).  
- Pour afficher la configuration de résolution de nom, au lieu de consulter le fichier `/etc/resolv.conf`, maintenant utiliser la commande :
```
resolvectl status
```
- Lister les connexions gérées par `NetworkManager` pour identifier celles sur lesquelles activer le *DNS over TLS* :
```
nmcli connection show
```
- Afficher le paramétrage d'une connexion, en remplaçant `<UUID>` par celui retourné dans la commande précédente :
```
nmcli connection show <UUID>
```
- Pour ajouter (en préfixant l'option par "`+`") ou supprimer (en préfixant l'option par "`-`") des serveurs DNS associés à une connexion (option `ipv4.dns` pour le protocole IPv4, et `ipv6.dns` pour le protocole IPv6) :
```
nmcli connection modify <UUID> +ipv4.dns 80.67.169.12 +ipv4.dns 80.67.169.40
```
- Pour chaque connexion ciblée, activer l'option `connection.dns-over-tls` (`-1`: par défaut, `0`: désactivé, `1`: [opportuniste](https://gitlab.freedesktop.org/NetworkManager/NetworkManager/-/issues/258#note_444510), `2`: activé) :
```
nmcli connection modify <UUID> connection.dns-over-tls 2
```
- Redémarrer le service `NetworkManager` pour prise en compte :
```
sudo systemctl restart NetworkManager.service
```
