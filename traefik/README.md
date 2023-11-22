# [Traefik](https://doc.traefik.io/traefik/) : _reverse proxy_ et _load balancer_
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
