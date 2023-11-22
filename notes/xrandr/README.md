# `xrandr`
## Ajouter une résolution personnalisée
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
