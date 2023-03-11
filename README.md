# Notes personnelles ([:earth_africa: web](https://mahikeul.github.io/))

## Grub : ajouter une entrée de démarrage de LiveCD

### Exemple avec partition racine fixe
- [Images debian avec firmwares non libres](https://cdimage.debian.org/images/unofficial/non-free/images-including-firmware/)
- Partition de boot de type `ext2`, table de partition `msdos`, première partition du premier disque (`(hd0,msdos1)`), image ISO déposée dans le sous-répertoire `images/`, noms des fichiers extraits de l'image (`*-5.10.0-18-amd64`)
```shell
 #!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "debian-live-11.6.0-amd64-xfce+nonfree" {
  insmod part_msdos
  insmod ext2
  set isofile="/images/debian-live-11.6.0-amd64-xfce+nonfree.iso"
  loopback loop (hd0,msdos1)$isofile
  linux (loop)/live/vmlinuz-5.10.0-18-amd64 boot=live findiso=$isofile
  initrd (loop)/live/initrd.img-5.10.0-18-amd64
}
```

### Version un peu plus dynamique
- Fichier `/etc/grub.d/50_boot_debian-live-11.5.0-amd64-xfce+nonfree`
- Image ISO LiveCD sur la partition de boot
```shell
#!/bin/sh
set -e

# Include the GRUB helper library for grub-mkconfig.
. /usr/share/grub/grub-mkconfig_lib

iso_file="/boot/images/debian-live-11.5.0-amd64-xfce+nonfree.iso"
iso_kernel_version="5.10.0-18-amd64"

echo "submenu 'Boot ISO Image: ${iso_file}' {"

submenu_indentation="$grub_tab"

echo "menuentry 'To RAM' {" | sed "s/^/$submenu_indentation/"

echo "insmod gzio" | grub_add_tab | sed "s/^/$submenu_indentation/"

if [ x$dirname = x/ ]; then
    if [ -z "${prepare_root_cache}" ]; then
        prepare_root_cache="$(prepare_grub_to_access_device ${GRUB_DEVICE} | grub_add_tab)"
    fi
    printf '%s\n' "${prepare_root_cache}" | sed "s/^/$submenu_indentation/"
else
    if [ -z "${prepare_boot_cache}" ]; then
        prepare_boot_cache="$(prepare_grub_to_access_device ${GRUB_DEVICE_BOOT} | grub_add_tab)"
    fi
    printf '%s\n' "${prepare_boot_cache}" | sed "s/^/$submenu_indentation/"
fi

sed "s/^/$submenu_indentation$grub_tab/" << EOF
set iso_file=$iso_file
loopback loop (${root})\$iso_file
linux (loop)/live/vmlinuz-$iso_kernel_version boot=live findiso=\$iso_file toram
initrd (loop)/live/initrd.img-$iso_kernel_version
EOF

echo "}" | sed "s/^/$submenu_indentation/"
echo "}"
```
