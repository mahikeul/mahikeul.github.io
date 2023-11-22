# [GitLab](https://docs.gitlab.com/)
## [GitLab Runner](https://docs.gitlab.com/runner/) : exécuter localement les _jobs_ d'une _pipeline_ GitLab
- Installation à partir des dépôts officiels GitLab pour Debian _trixie_ ([&#x21aa; source](https://docs.gitlab.com/runner/install/linux-repository.html)):
```shell
# Configuration de apt
curl -L "https://packages.gitlab.com/install/repositories/runner/gitlab-runner/script.deb.sh" | sudo bash
```
```shell
# Installation de la dernière version
sudo apt-get install gitlab-runner
```
```shell
# Pour installer une version différente de la dernière
# - Liste des versions disponibles
apt-cache madison gitlab-runner
# - Installation de la version choisie
sudo apt-get install gitlab-runner=15.11.1
```
- Pour utiliser `podman` plutôt que `docker` comme _executor_ ([&#x21aa; source](https://docs.gitlab.com/runner/executors/docker.html#use-podman-to-run-docker-commands)), ajouter la ligne suivante dans le fichier de configuration des _runners_ `config.toml` dans la section `[runners.docker]` (`1000` étant l'UID Unix de l'utilisateur) :
```toml
host = "unix:///run/user/1000/podman/podman.sock"
```
- Pour pouvoir utiliser le sous-système de conteneur lors de l'exécution d'un runner gitlab de type _executor:docker_ (utilisation de la commande `docker` grâce à une image `dind`), une authorisation est nécessaire ([&#x21aa; source](https://docs.gitlab.com/runner/executors/docker.html#privileged-mode)); il faut rajouter la ligne suivante dans le fichier de configuration des _runners_ `config.toml` dans la section `[runners.docker]` :
```toml
privileged = true
```
