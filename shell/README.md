# Shell
## Commande `bash` pour comparer 2 listes de variables d'environnement extrait de 2 fichiers différents
```shell
diff <(sed -n 's/^.*\${\([^}\:]*\)\(\:[^}\:]*\)\?}.*$/\1/p' application.yml | sort -u) <(yq -r '.env | keys | @tsv' values.yaml | sed 's/\t/\n/g' | sort)
```
- `yq`: [Command-line YAML/XML/TOML processor - jq wrapper for YAML, XML, TOML documents](https://github.com/kislyuk/yq).
- `application.yml`: fichier YAML (configuration SpringBoot par exemple) dans lequel sont déclarés des variables d'environnement sous la forme `${...}`.
- `values.yaml`: fichier YAML (variables Helm par exemple) dans lequel sont listées des variables d'environnements (`nom`:`valeur`) sous l'arborescence `env`.
## Valoriser un fichier `/var/www/config.json` contenant des variables d'environnement pour être servi ensuite par *Apache*
- Commande utilisée : `envsubst` (`apt install gettext-base`, [&#x21aa; manpage](https://manpages.debian.org/testing/gettext-base/envsubst.1.en.html))
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
## Retrouver le package Debian ayant amené un fichier
Différents moyens ([&#x21aa; source](https://linuxhint.com/find-debian-package-provides-file/)) :
```shell
dpkg -S /path/to/file
```
```shell
dpkg-query -S '/path/to/file'
```
ou, après les commandes `sudo apt install apt-file` et `sudo apt-file update` :
```shell
apt-file search /path/to/file 
```
