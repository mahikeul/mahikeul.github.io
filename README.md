# Notes personnelles
## [Grub : ajouter une entrée pour démarrer un LiveCD](grub-entry-livecd/README.md)
## [Mise en place d'un environnement Kubernetes local](k8s-local/README.md)
## [Activer le DNS over TLS](dns-over-tls/README.md)
## Vim : copy to clipboard
`vim --version` : `+xterm_clipboard` needed ([&#x21aa; source](https://stackoverflow.com/a/14225889))
## NetworkManager
- Pour qu'une connexion (_VPN_ par exemple) ne soit jamais la _route_ par défaut (pour pouvoir accéder à Internet si la connexion cible un réseau fermé) :
```shell
nmcli connection modify <UUID> ipv4.never-default true
```
- Pour avoir des informations sur un paramètre d'une connexion, passer en mode édition avec la commande suivante sur la connexion `<UUID>` visée, puis invoquer `describe {nom-du-paramètre}` (par exemple, `describe ipv4.never-default`) :
```shell
nmcli connection edit <UUID>
```
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
## Quelques containeurs utiles
- Pour installer `podman`, voir l'article [k8s local](k8s-local/README.md).
- [Cockpit](), pour gérer visuellement les containeurs (entre autres), par défaut à l'adresse `http://localhost:9090` :
```shell
sudo apt install cockpit-podman
```
- [PlantUML](https://plantuml.com/) ([&#x21aa; docker hub](https://hub.docker.com/r/plantuml/plantuml-server)), pour générer localement les diagrammes dans [VSCode](https://open-vsx.org/extension/jebbs/plantuml) :
```shell
podman run -p 9001:8080 --name plantuml-server docker.io/plantuml/plantuml-server:jetty
```
- [Kroki](https://kroki.io/) ([&#x21aa; docker hub](https://hub.docker.com/r/yuzutech/kroki)), pour générer localement les diagrammes avec l'extension navigateur [Markdown Diagrams](https://chrome.google.com/webstore/detail/markdown-diagrams/pmoglnmodacnbbofbgcagndelmgaclel) :
```shell
podman run -p 9005:8000 --name kroki docker.io/yuzutech/kroki
```
- [Swagger-UI](https://swagger.io/tools/swagger-ui/) ([&#x21aa; docker hub](https://hub.docker.com/r/swaggerapi/swagger-ui)), quand l'UI n'est pas intégrée au microservice :
```shell
podman run -p 9002:8080 --name swagger-ui docker.io/swaggerapi/swagger-ui
```
## [VSCodium](https://github.com/VSCodium/vscodium/) : VS Code sans télémétrie, ni extension propriétaire
- Installation :
```shell
sudo apt install extrepo
sudo extrepo enable vscodium
sudo apt update
sudo apt install codium
```
- Extensions utilisées ([&#x21aa; magasin *OpenSource* par défaut](https://open-vsx.org/)) :
  - [kubernetes](https://open-vsx.org/extension/ms-kubernetes-tools/vscode-kubernetes-tools) : nécessite `kubectl`, `docker` ou `buildah`, `helm` (pour installation, voir article [k8s local](k8s-local/README.md)).
  - [PlantUML](https://open-vsx.org/extension/jebbs/plantuml) : utilisation locale de [`plantuml-server`](https://github.com/plantuml/plantuml-server) recommandée (voir l'article [Quelques containeurs utiles](#quelques-containeurs-utiles)).
## [Barrier](https://github.com/debauchee/barrier) : partage de clavier/souris/presse-papier entre ordinateurs
- Installation :
```shell
sudo apt install barrier
```
- Configuration serveur ([&#x21aa; mise en place du certificat SSL](https://stackoverflow.com/questions/67343804/error-ssl-certificate-doesnt-exist-home-rsvay-snap-barrier-kvm-2-local-shar)) :
```shell
cd ${HOME}/.local/share/barrier/SSL/
openssl req -x509 -nodes -days 365 -subj /CN=barrier -newkey rsa:4096 -keyout Barrier.pem -out Barrier.pem
```
- Configuration client (SSL activé) : accepter manuellement le *fingerprint* lors de la connexion, ou le générer sur le serveur et déplacer le fichier sur le client dans le répertoire `${HOME}/.local/share/barrier/SSL/Fingerprints` (ou copier son contenu dans le fichier `TrustedServers.txt` dans ce même répertoire).
```shell
openssl x509 -fingerprint -sha256 -noout -in Barrier.pem > Fingerprints/Local.txt
sed -e "s/.*=/v2:sha256:/" -i Fingerprints/Local.txt
```
## [Traefik](https://doc.traefik.io/traefik/), _reverse proxy_ et _load balancer_
- Fichier de configuration :
```shell
curl https://raw.githubusercontent.com/traefik/traefik/v2.10/traefik.sample.yml -o traefik.sample.yml
```
- Copie du fichier vers `traefik.yml` + lignes à décommenter :
```yaml
api:
  insecure: true
providers:
  docker:
     exposedByDefault: true
```
- Démarrage (avec `docker`):
```shell
docker run -d -p 8080:8080 -p 80:80 -p 443:443 -v $(pwd)/traefik.yml:/etc/traefik/traefik.yml -v /var/run/docker.sock:/var/run/docker.sock --name traefik traefik
```
- Accès au tableau de bord : http://localhost:8080/dashboard
## `xrandr` : ajouter une résolution personnalisée
- Génération d'une _modeline_ avec `cvt` (alternative ? `gtf`). Par exemple, paramétrer une résolution pour pouvoir faire du _PBP_ (images côte-à-côte) sur son écran 4K avec un taux de rafraichissement de 30Hz (parce que l'entrée `HDMI-1` de votre écran ne supporte pas un taux plus élevé dans une résolution supérieure à `1920x1080`):
```shell
cvt 1920 2180 30
```
- La commande précédente doit donner quelque chose qui ressemble à :
```
# 1920x2160 29.96 Hz (CVT) hsync: 65.92 kHz; pclk: 168.75 MHz
Modeline "1920x2160_30.00"  168.75  1920 2040 2240 2560  2160 2163 2173 2200 -hsync +vsync
```
- Maintenant, il faut indiquer à `xrandr` la configuration de cette nouvelle _modeline_ (copier/coller de la dernière ligne sans `ModeLine`) :
```shell
xrandr --newmode "1920x2160_30"  168.75  1920 2040 2240 2560  2160 2163 2173 2200 -hsync +vsync
```
- On peut maintenant ajouter (mais pas appliquer) cette nouvelle résolution (nommée `1920x2160_30`) à celles disponibles pour une sortie spécifiée (ici `HDMI-1`) :
```shell
xrandr --addmode HDMI-1 1920x2160_30
```
- Les différentes sorties sont à retrouver avec la simple commande `xrandr`.  
  Pour les sorties actives : `xrandr --listmonitors`
- Pour appliquer en ligne de commande la résolution nommée `1920x2160_30` à la sortie `HDMI-1` :
```shell
xrandr --output HDMI-1 --mode 1920x2160_30
```
## [nvm](https://github.com/nvm-sh/nvm) : Node Version Manager
- Installation par `git` ([&#x21aa; source](https://github.com/nvm-sh/nvm#git-install)).  
Récupération des sources :
```shell
git clone https://github.com/nvm-sh/nvm.git
```
- Si non cloné dans le répertoire `~/.nvm`, créer un lien :
```shell
ln -s $(pwd)/nvm ~/.nvm
```
- Se placer dans le répertoire (`cd ~/.nvm`) et pointer sur la dernière version :
```shell
git checkout `git describe --abbrev=0 --tags --match "v[0-9]*" $(git rev-list --tags --max-count=1)`
```
- Activation de la version pointée :
```shell
. ./nvm.sh
```
- Activation de la complétion `bash` :
```shell
. ./bash_completion
```
- Lignes à rajouter à `~/.bashrc` pour que les 2 activations précédentes soient prises en compte au login :
```shell
export NVM_DIR="$HOME/.nvm"
[ -s "$NVM_DIR/nvm.sh" ] && \. "$NVM_DIR/nvm.sh"
[ -s "$NVM_DIR/bash_completion" ] && \. "$NVM_DIR/bash_completion"
```
- Pour mettre à-jour `nvm`, synchroniser l'historique `git` avec la commande suivante dans le répertoire `cd ${NVM_DIR}`, pour ensuite pointer sur la dernière version avec la commande `git checkout` décrite plus haut, suivi d'un `source ~/.bashrc` (prise en compte des activations) :
```shell
git fetch --tags origin
```
- Lister les principales versions de `node` disponibles :
```shell
nvm ls
```
- Installer une version (ici, la dernière version de la branche `14`) :
```shell
nvm install 14
```
- Pour spécifier une version de travail (ici, la version correspondant à la branche `14`) :
```shell
nvm use 14
```
