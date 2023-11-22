# [Barrier](https://github.com/debauchee/barrier) : partage de clavier/souris/presse-papier entre ordinateurs
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