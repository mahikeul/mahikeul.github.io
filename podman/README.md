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
