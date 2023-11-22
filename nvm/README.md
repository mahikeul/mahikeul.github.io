# [nvm](https://github.com/nvm-sh/nvm) : Node Version Manager
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
