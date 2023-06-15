***Notes personnelles***
---
<br />

# Kubernetes : podman (possibly rootless)
- k3d : https://k3d.io/v5.4.2/usage/advanced/podman/
- k3s : 
- kind : https://kind.sigs.k8s.io/docs/user/rootless/ | https://podman-desktop.io/docs/kubernetes/kind (WSL)
- minikube : https://minikube.sigs.k8s.io/docs/drivers/podman/
- inside kubernetes : https://www.redhat.com/sysadmin/podman-inside-kubernetes

# Vim : copy to clipboard
- `vim --version` : `+xterm_clipboard` needed :  https://stackoverflow.com/a/14225889

# [Grub : ajouter une entrée pour démarrer un LiveCD](1/README.md)

# [Mise en place d'un environnement Kubernetes local](2/README.md)

# [Activer le DNS over TLS](3/README.md)

# Commande `bash` pour comparer 2 listes de variables d'environnement extrait de 2 fichiers différents
```shell
diff <(sed -n 's/^.*\${\([^}\:]*\)\(\:[^}\:]*\)\?}.*$/\1/p' application.yml | sort -u) <(yq -r '.env | keys | @tsv' values.yaml | sed 's/\t/\n/g' | sort)
```
- `yq`: [Command-line YAML/XML/TOML processor - jq wrapper for YAML, XML, TOML documents](https://github.com/kislyuk/yq).
- `application.yml`: fichier YAML (configuration SpringBoot par exemple) dans lequel sont déclarés des variables d'environnement sous la forme `${...}`.
- `values.yaml`: fichier YAML (variables Helm par exemple) dans lequel sont listées des variables d'environnements (`nom`:`valeur`) sous l'arborescence `env`.
# Valoriser un fichier `/var/www/config.json` contenant des variables d'environnement pour être servi ensuite par *Apache*
- Commande utilisée : `envsubst` ([manpages](https://manpages.debian.org/testing/gettext-base/envsubst.1.en.html))
- Activer les modules nécessaires : `a2enmod alias cgi env`.  
  Le module `cgi` devrait être configuré dans le fichier de configuration `/etc/apache2/conf-available/serve-cgi-bin.conf`.
- Créer le script `config.cgi` dans le répertoire `/usr/lib/cgi-bin/` (répertoire configuré dans le fichier `serve-cgi-bin.conf`) :
```shell
#!/bin/sh

cat << EOL
Content-type: application/json

$(cat /var/www/config.json | envsubst)
EOL
```
- Créer le fichier de configuration permettant de passer les variables d'environnement nécessaires à *Apache* :
```shell
sed -n 's/^.*\${\([^}\:]*\)\(\:[^}\:]*\)\?}.*$/PassEnv \1/p' /var/www/config.json > /etc/apache2/conf-enabled/passenv.conf
```
- Par l'URL `/cgi-bin/config.cgi` on obtient l'équivalent valorisé du fichier `/var/www/config.json`.
# Retrouver le package Debian ayant amené un fichier
([*source*](https://linuxhint.com/find-debian-package-provides-file/))
```shell
dpkg -S /path/to/file
```
ou
```shell
dpkg-query -S '/path/to/file'
```
ou
```shell
sudo apt install apt-file && sudo apt-file update
apt-file search /path/to/file 
```
