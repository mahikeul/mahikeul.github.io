# Notes personnelles

## Kubernetes : podman (possibly rootless)
- k3d : https://k3d.io/v5.4.2/usage/advanced/podman/
- k3s : 
- kind : https://kind.sigs.k8s.io/docs/user/rootless/ | https://podman-desktop.io/docs/kubernetes/kind (WSL)
- minikube : https://minikube.sigs.k8s.io/docs/drivers/podman/
- inside kubernetes : https://www.redhat.com/sysadmin/podman-inside-kubernetes

## Vim : copy to clipboard
- `vim --version` : `+xterm_clipboard` needed :  https://stackoverflow.com/a/14225889

## Grub : ajouter une entrée pour démarrer un LiveCD

### Exemple avec partition racine fixe
- [Images Debian avec firmwares non libres](https://cdimage.debian.org/images/unofficial/non-free/images-including-firmware/)
- Partition de boot de type `ext2`, table de partition `msdos`, première partition du premier disque (`(hd0,msdos1)`), image ISO déposée dans le sous-répertoire `images/`, noms des fichiers (kernel et initrd) extraits de l'image (`*-5.10.0-18-amd64`)
```shell
 #!/bin/sh
exec tail -n +3 $0
# This file provides an easy way to add custom menu entries.  Simply type the
# menu entries you want to add after this comment.  Be careful not to change
# the 'exec tail' line above.

menuentry "debian-live-11.6.0-amd64-xfce+nonfree" {
  insmod part_msdos
  insmod ext2
  set isofile="/images/debian-live-11.5.0-amd64-xfce+nonfree.iso"
  loopback loop (hd0,msdos1)$isofile
  linux (loop)/live/vmlinuz-5.10.0-18-amd64 boot=live findiso=$isofile
  initrd (loop)/live/initrd.img-5.10.0-18-amd64
}
```

### Version _dynamique_ avec une seule entrée
- Image ISO LiveCD **Debian** sur la partition de boot.
- Extraction automatique des noms des fichiers de boot (via `mount $file $dir -o loop -r`).
- Avec [désactivation du module TPM](https://askubuntu.com/a/1244886) (via `rmmod tpm`).
```shell
#!/bin/sh
set -e

# Include the GRUB helper library for grub-mkconfig.
. /usr/share/grub/grub-mkconfig_lib

iso_entry () {
    iso_file="$1"
    loop_options="$2"

    echo "Adding boot menu entry for LiveCD $iso_file $loop_options" >&2

    # mount iso file to extract kernel et initrd filenames
    iso_tmpdir=$(mktemp -d)
    mount $iso_file $iso_tmpdir -o loop -r
    cd $iso_tmpdir
    iso_kernel=$(find live -iname "vmlinuz*")
    iso_initrd=$(find live -iname "initrd*")
    cd - > /dev/null
    umount $iso_tmpdir
    rmdir $iso_tmpdir

    echo "menuentry 'Boot $(basename $iso_file) $loop_options' {"

    sed "s/^/$grub_tab/" << EOF

$(prepare_grub_to_access_device "$($grub_probe -t device "$iso_file")")

rmmod tpm
load_video
insmod gzio

loopback loop $iso_file
linux (loop)/$iso_kernel findiso=$iso_file $loop_options
initrd (loop)/$iso_initrd
EOF

    echo "}"
}

iso_entry "/boot/images/debian-live-11.6.0-amd64-xfce+nonfree.iso" "boot=live toram"
```

### Version brouillon _semi-dynamique_ avec sous-menus et _debug_
- Fichier `/etc/grub.d/50_boot_debian-live-11.5.0-amd64-xfce+nonfree`
- Image ISO LiveCD **Debian** sur la partition de boot `/boot/images/debian-live-11.5.0-amd64-xfce+nonfree.iso`
- Extraction manuelle de la version du kernel ̀`5.10.0-18-amd64`
```shell
#!/bin/sh
set -e

# Include the GRUB helper library for grub-mkconfig.
. /usr/share/grub/grub-mkconfig_lib

iso_entry () {
    loop_title="$1"
    loop_options="$2"

    echo "menuentry '$loop_title' {" | sed "s/^/$submenu_indentation/"

    sed "s/^/$submenu_indentation$grub_tab/" << EOF
rmmod tpm
load_video
insmod gzio

EOF

    if [ x$dirname = x/ ]; then
        target_device="${GRUB_DEVICE}"

        if [ -z "${prepare_root_cache}" ]; then
            prepare_root_cache="$(prepare_grub_to_access_device ${GRUB_DEVICE} | grub_add_tab)"
        fi
        printf '%s\n' "${prepare_root_cache}" | sed "s/^/$submenu_indentation/"
    else
        target_device="${GRUB_DEVICE_BOOT}"

        if [ -z "${prepare_boot_cache}" ]; then
            prepare_boot_cache="$(prepare_grub_to_access_device ${GRUB_DEVICE_BOOT} | grub_add_tab)"
        fi        
        printf '%s\n' "${prepare_boot_cache}" | sed "s/^/$submenu_indentation/"
    fi

    if [ -z "$fs_uuid" ]; then
        fs_uuid="$("${grub_probe}" --device $target_device --target=fs_uuid 2> /dev/null)"
    fi

    if [ -z "$partmap" ]; then
        partmap="$("${grub_probe}" --device $target_device --target=partmap 2> /dev/null)"
    fi

    if [ -z "$fs" ]; then
        fs="$("${grub_probe}" --device $target_device --target=fs 2> /dev/null)"
    fi

    if [ -z "$fs_hint" ]; then
        fs_hint="$("${grub_probe}" --device $target_device --target=compatibility_hint 2> /dev/null)"
    fi

    if [ -z "$hints" ]; then
        hints="$("${grub_probe}" --device $target_device --target=hints_string 2> /dev/null)"
    fi

    if [ -z "$abstraction" ]; then
        abstraction="$("${grub_probe}" --device $target_device --target=abstraction 2> /dev/null)"
    fi

    if [ -z "$device" ]; then
        device="$("${grub_probe}" --device $target_device --target=device 2> /dev/null)"
    fi

    if [ -z "$disk" ]; then
        disk="$("${grub_probe}" --device $target_device --target=disk 2> /dev/null)"
    fi

    if [ -z "$drive" ]; then
        drive="$("${grub_probe}" --device $target_device --target=drive 2> /dev/null)"
    fi

    if [ -z "$gpt_parttype" ]; then
        gpt_parttype="$("${grub_probe}" --device $target_device --target=gpt_parttype 2> /dev/null)"
    fi

    if [ -z "$msdos_parttype" ]; then
        msdos_parttype="$("${grub_probe}" --device $target_device --target=msdos_parttype 2> /dev/null)"
    fi

    if [ -z "$partuuid" ]; then
        partuuid="$("${grub_probe}" --device $target_device --target=partuuid 2> /dev/null)"
    fi

    sed "s/^/$submenu_indentation$grub_tab/" << EOF

echo "target_device=$target_device"
echo "fs_uuid=$fs_uuid"
echo "partmap=$partmap"
echo "fs=$fs"
echo "fs_hint=$fs_hint"
echo "hints=$hints"
echo "abstraction=$abstraction"
echo "device=$device"
echo "disk=$disk"
echo "gpt_parttype=$gpt_parttype"
echo "msdos_parttype=$msdos_parttype"
echo "partuuid=$partuuid"

set iso_file="$iso_file"

echo "loopback loop $iso_file"
loopback loop \$iso_file

echo "linux (loop)/live/vmlinuz-$iso_kernel_version findiso=$iso_file $loop_options"
linux (loop)/live/vmlinuz-$iso_kernel_version findiso=\$iso_file $loop_options

echo "initrd (loop)/live/initrd.img-$iso_kernel_version"
initrd (loop)/live/initrd.img-$iso_kernel_version
EOF

    echo "}" | sed "s/^/$submenu_indentation/"
}

iso_file="/boot/images/debian-live-11.5.0-amd64-xfce+nonfree.iso"
iso_kernel_version="5.10.0-18-amd64"

echo "submenu 'Boot ISO Image: ${iso_file}' {"

submenu_indentation="$grub_tab"

iso_entry "Normal" "boot=live "
iso_entry "To RAM" "boot=live toram"

echo "}"
```
