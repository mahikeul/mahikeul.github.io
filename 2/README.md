***Environnement Kubernetes local***
---
<br />

# Mise en place de l'infrastructure, avec `Kind`
* Alternatives :
  * [`k3d`](https://k3d.io/v5.4.2/usage/advanced/podman/)
  * `k3s`
  * [`minikube`](https://minikube.sigs.k8s.io/docs/drivers/podman/)
* [Installation de `kind`](https://kind.sigs.k8s.io/docs/user/quick-start/).
* Rootless :
  * avec [Linux](https://kind.sigs.k8s.io/docs/user/rootless/), et [Docker](https://docs.docker.com/go/rootless/) ou [Podman](https://github.com/containers/podman/blob/master/docs/tutorials/rootless_tutorial.md)
  * avec [WSL et Podman](https://podman-desktop.io/docs/kubernetes/kind)
* [Configuration avancée pour la création d'un cluster](https://kind.sigs.k8s.io/docs/user/configuration/)
    
## Exemple sous Debian Testing (`12` *bookworm*) en *rootless* avec Podman
* Installation de `kubectl` (`1.27.1`) (doc sur [kubernetes.io](https://kubernetes.io/fr/docs/tasks/tools/install-kubectl/#installation-%C3%A0-l-aide-des-gestionnaires-des-paquets-natifs)) :
```shell
sudo apt-get update && sudo apt-get install apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
echo "deb https://apt.kubernetes.io/ kubernetes-xenial main" | sudo tee -a /etc/apt/sources.list.d/kubernetes.list
sudo apt-get update && sudo apt-get install kubectl
```
* Installation de podman ([`4.3.1`](https://packages.debian.org/testing/podman)), slirp4netns ([`1.2.0`](https://packages.debian.org/testing/slirp4netns)), fuse-overlayfs ([`1.10`](https://packages.debian.org/testing/fuse-overlayfs)) et golang ([`1.19`](https://packages.debian.org/testing/golang))  
  (pour installer la version de `kubectl` de la distribution, ajouter [`kubernetes-client`](https://packages.debian.org/testing/kubernetes-client)) :
```shell
sudo apt-get install podman slirp4netns fuse-overlayfs golang
```
* Installation de kind (`latest` = `v0.20.0`) (autres méthodes d'installation et dernière version dans la [doc](https://kind.sigs.k8s.io/docs/user/quick-start/#installing-with-go-install))  
  (`${HOME}/go/bin` doit être dans le `$PATH` pour que la commande `kind` soit ensuite disponible: `sudo micro /etc/profile`) :
```shell
go install sigs.k8s.io/kind@latest
```
* Variable d'environnement à modifier pour que `kind` utilise `podman`  
  (par exemple, pour une prise en compte globale, l'ajouter au fichier `/etc/environment`) :
```shell
KIND_EXPERIMENTAL_PROVIDER=podman
```
* Création simple d'un cluster nommé `vasco`  
  (la version de K8s venant par défaut avec `kind` `0.20.0` est la `1.27.3`, d'autres images disponibles dans les [notes de version](https://github.com/kubernetes-sigs/kind/releases)).  
  Sans l'option `--name {cluster}`, le nom du cluster est `kind`.  
  Si un nom est paramétré à la création, cette option (`--name {cluster}`) devra être ajoutée à toute commande `kind`.
```shell
kind create cluster --name vasco
```
* Suppression du cluster :
```shell
kind delete cluster --name vasco
```
* Pour avoir des informations sur le cluster (le contexte Kubernetes créé par `kind` étant `kind-{cluster-name}`) :
```shell
kubectl cluster-info --context kind-vasco
```
* Fichier de configuration du cluster (si `$KUBECONFIG` vide) : `${HOME}/.kube/config`
* Liste des clusters créés par `kind`:
```shell
kind get clusters
```
* Liste des noeuds dans un cluster (ici `vasco`, créé plus haut) :
```shell
kind get nodes --name vasco
```
* Liste des images (`crictl images`), des containeurs (`crictl ps`) ou des pods (`crictl pods`), présents sur un noeud, en reprenant le nom du noeud (`vasco-control-plane`) de la commande précédente :
```shell
podman exec -it vasco-control-plane crictl images
podman exec -it vasco-control-plane crictl ps
podman exec -it vasco-control-plane crictl pods
```
* Pour exporter tous les logs du cluster dans un répertoire temporaire :
```shell
kind export logs --name vasco
```

# Déploiement du [Dashboard Kubernetes](https://kubernetes.io/docs/tasks/access-application-cluster/web-ui-dashboard/)
* En ligne de commande, pour avoir la liste des *namespace*, *node*, *pods*, *endpoints*, *service*, *service account* et *secret* Kubernetes de tous les *namespace*  
(pour avoir la liste de toutes les ressources interrogeables : `kubectl api-resources`) :
```shell
kubectl get --all-namespaces ns,no,po,ep,svc,sa,secrets
```
* Pour déployer le tableau de bord en version `2.7.0` (voir les dernières [releases](https://github.com/kubernetes/dashboard/releases)) :
```shell
kubectl apply -f https://raw.githubusercontent.com/kubernetes/dashboard/v2.7.0/aio/deploy/recommended.yaml
```
* Par défaut, le tableau de bord sera déployé dans le *namespace* `kubernetes-dashboard` : toute commande `kubectl` ciblant l'environnement du tableau de bord devra inclure l'option `-n kubernetes-dashboard` ciblant ce *namespace*.
* Activation du proxy pour accéder au tableau de bord (pour ne pas utiliser de proxy, voir [ici](https://kubernetes.io/docs/tasks/administer-cluster/access-cluster-api/#without-kubectl-proxy)) :
```shell
kubectl proxy
```
* Lien d'accès au tableau de bord (si le proxy sert `localhost:8001`) : [kubernetes-dashboard](http://localhost:8001/api/v1/namespaces/kubernetes-dashboard/services/https:kubernetes-dashboard:/proxy/)
* Création d'un compte de service `admin-user` dans le *namespace* du tableau de bord, dans le but de récupérer son *token* :
```shell
kubectl -n kubernetes-dashboard create sa admin-user
```
* Commande pour associer le rôle `cluster-admin` au compte de service `admin-user`, pour accéder à toutes les resources du cluster :
```shell
kubectl -n kubernetes-dashboard create clusterrolebinding admin-user --clusterrole=cluster-admin --serviceaccount=kubernetes-dashboard:admin-user
```
* Commande pour générer le token à partir du compte de service (détails : [creating-sample-user](https://github.com/kubernetes/dashboard/blob/master/docs/user/access-control/creating-sample-user.md))  
(si la commande `token` n'est pas disponible, c'est que la version de `kubectl` n'est pas assez récente (< `1.24` ?). Pistes pour contourner le problème : [long live API token](https://kubernetes.io/docs/tasks/configure-pod-container/configure-service-account/#manually-create-a-long-lived-api-token-for-a-serviceaccount), [install Dashboard v2.2.0](https://adamtheautomator.com/kubernetes-dashboard/), [obtening service account token](https://docs.selectel.com/cloud/managed-kubernetes/instructions/service-account-token/)) :
```shell
kubectl -n kubernetes-dashboard create token admin-user
```

# Mise en place de [Helm](https://helm.sh/docs/intro/quickstart/), package manager for Kubernetes
* [Installation](https://helm.sh/docs/intro/install/)
* [Packages disponibles](https://artifacthub.io/packages/search)

# Pour aller plus loin
* [How to use Podman inside of Kubernetes](https://www.redhat.com/sysadmin/podman-inside-kubernetes)
