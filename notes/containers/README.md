# Quelques conteneurs utiles
- Pour installer `podman`, voir l'article [k8s local](../k8s-local/README.md).
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
