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
  - [PlantUML](https://open-vsx.org/extension/jebbs/plantuml) : utilisation locale de [`plantuml-server`](https://github.com/plantuml/plantuml-server) recommandée (voir l'article [Quelques containeurs utiles](../containers/README.md)).
