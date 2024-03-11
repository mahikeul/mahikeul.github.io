# [Podman](https://podman.io)
## Quelques commande utiles
- Supprime :
> - all stopped containers
> - all networks not used by at least one container
> - all images without at least one container associated with them
> - all build cache
```shell
podman system prune --all
```
- Supprime une ou plusieurs images locales (identique à `podman image rm`)
```shell
podman rmi [-a,--all] [-f,--force] [-i,--ignore] [--no-prune] [<image>]
```
- Monter une image (`podman unshare` nécessaire en mode _rootless_) :
```shell
podman unshare podman image mount <image>
```
## Migration d'image Docker ([&#x21aa; source](https://www.devopszones.com/2022/02/how-to-migrate-from-docker-containers.html))
- Faire une copie du conteneur vers une nouvelle image :
```shell
docker commit <containerId | containerName> <imagename:tag_$(date +%s)>
```
- Sauvegarde de l'image :
```shell
docker save <imagename[:tag]> | gzip > imagename_tag_$(date +%s).tar.gz
```
- Chargement de l'image sauvegardée par podman :
```shell
podman load < imagename_tag_timestamp.tar.gz
```
- L'image `localhost/imagename:tag_timestamp` doit alors apparaitre dans la liste :
```shell
podman images
```
ou
```shell
podman image ls
```
- Lister les montages attachés au conteneur docker et repérer les types `bind` :
```shell
docker container inspect <containerId | containerName> -f "{{ json .Mounts }}" | jq
```
- Arrêter les conteneurs et copier les répertoires `Source` (noter la valeur de `Destination`) :
```shell
sudo cp -r /var/lib/docker/volumes/<sourcePath> ${HOME}/.local/share/containers/storage/volumes/<sourcePath>
```
- Modifier le propriétaire des fichiers copiés :
```shell
sudo chown -R <user>:<group> /home/<user>/.local/share/containers/storage/volumes/<sourcePath>
```
- L'image peut ensuite être lancée comme toute autre image :
```shell
podman run [--rm] [--name <imagename>] [-v ${HOME}/.local/share/containers/storage/volumes/<sourcePath>:<destinationPath>] localhost/<imagename[:tag]>
```
