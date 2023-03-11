# Notes personnelles [:earth_africa:web](https://mahikeul.github.io/)

## Grub : ajouter une entrée de démarrage de LiveCD

Exemple :
- [images avec firmwares non libres](https://cdimage.debian.org/images/unofficial/non-free/images-including-firmware/)
```
menuentry "debian-live-11.6.0-amd64-xfce+nonfree" {
  insmod part_msdos
  insmod ext2
  set isofile="/images/debian-live-11.6.0-amd64-xfce+nonfree.iso"
  loopback loop (hd0,msdos1)$isofile
  linux (loop)/live/vmlinuz-5.10.0-18-amd64 boot=live findiso=$isofile
  initrd (loop)/live/initrd.img-5.10.0-18-amd64
}
```
