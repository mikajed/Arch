2. Tastatur auf Deutsch stellen: `loadkeys de-latin1`
3. Schauen ob das Netz geht: `ip a`
4. Zeit einstellen: `timedatectl set-ntp true`

# Festplatten/Partition/Formatierung und Mounting

6. Partitionieren: `lsblk` zum schauen, welche Festplatten vorhanden sind.
7. Danach: `fdisk /dev/sda`
8. Jetzt `g` eingeben da es eine EFI installation sein soll
9. Neue Partition erstellen mit: `n` für neu.
10. Partition 1 als default lassen
11. Erster Sektor auch einfach die Defaults lassen
12. Im letzten Schritt soll die Partition 512mb haben: "+512MB"
13. Partition 2 (root) erstellen: `n`
14. Partition 2 default lassen und enter drücken.
15. Ersten Sektor als default lassen und enter drücken.
16. Im letzten Schritt soll die Partition 50gb haben: "+50G" und enter.
    1. Partition 3 (Home) erstellen: `n` + enter und alle Sektoren default lassen
17. Partitionen sind jetzt vorhanden: `w` und enter.
18. `lsblk`

# Partitionen erstellen:

20. Erste Partition (EFI deshalb fat): `mkfs.vfat /dev/sda1`
21. Zweite Partition: `mkfs.ext4 /dev/sda2`
22. Dritte Partition: `mkfs.ext4 /dev/sda3`

# Partitionen mounten

23. Erstmal wo das System sein wird: `mount /dev/sda2 /mnt`
24. Die EFI in die Bootpartition mounten: `mkdir /mnt/boot`
25. `mkdir /mnt/boot/EFI`
26. sda1 in das EFI mounten:

    1.  `mount /dev/sda1 /mnt/boot/EFI`
    2.  `mkdir /mnt/home`
    3.  `mount /dev/sda3 /mnt/home`

27. ### ohne 3 home partition:
    1. _`mount /dev/sda2 /mnt`_
    2. _`mkdir /mnt/boot`_
    3. _`mkdir /mnt/boot/EFI`_
    4. _`mount /dev/sda1 /mnt/boot/EFI`_
28. `lsblk`

# Basis Installation:

28. `pacstrap -K /mnt base sudo linux linux-firmware nano amd-ucode/indel-ucode`

# FSTAB Datei generieren:

31. `genfstab -U /mnt >> /mnt/etc/fstab`
32. Überprüfen ob alle Partitionen da sind: `cat /mnt/etc/fstab`

# Installer verlassen um im System Einstellungen zu machen:

34. Installer verlassen mit: `arch-chroot /mnt`
35. Swapfile: `dd if=/dev/zero of=/swapfile bs=1M count=2048 status=progress`
36. Zugriffsrechte ändern: `chmod 600 /swapfile`
37. Swap-File erstellen: `mkswap /swapfile`
38. Swap aktivieren: `swapon /swapfile`
39. Swap in fstab verschieben: `nano /etc/fstab`
40. Ganz unten schreiben: `/swapfile none swap defaults 0 0`

# Zeitzone einrichten:

42. `ln -sf /usr/share/zoneinfo/Europe/Berlin /etc/localtime`
43. Uhr synchronisieren: `hwclock --systohc`

# Spracheinstellungen:

44. `nano /etc/locale.gen`
45. Locale erstellen: `locale-gen`
46. Locale konfigurieren: `nano /etc/locale.conf`
47. `LANG=de_DE.UTF-8` einfügen.
48. `/etc/vconsole.conf` --- `KEYMAP=de-latin1`

# Hostnamen einstellen:

50. `nano /etc/hostname` - Namen eingeben z.B. "arch"

# Hosts File erstellen:

51. `nano /etc/hosts`
52. Dort dann:
    1.  `127.0.0.1` tab `localhost`
    2.  `::1`     tab `localhost`
    3.  `127.0.1.1` tab `arch.localdomain` tab `arch`

# Root Passwort erstellen:

54. `passwd`

# Bootloader installieren:

56. `pacman -S grub efibootmgr networkmanager mtools dosfstools linux-headers`
57. `grub-install --target=x86_64-efi --efi-directory=/boot/EFI --bootloader-id=GRUB`
58. `grub-mkconfig -o /boot/grub/grub.cfg`

# Internet aktivieren:

61. `systemctl enable NetworkManager` damit das Internet beim Boot startet. (bei WLAN noch weitere configs)

# Neuen Benutzer anlegen:

62. `useradd -m -G wheel michael` + enter
63. `passwd michael` - danach Passwort eingeben.
64. `EDITOR=nano visudo`
65. Hier `%wheel ALL=(ALL) ALL`  -  uncommenten
66. mit `exit` beenden.

# Partitionen unmounten:

69. `umount -R /mnt`
70. `reboot`
71. danach als "root" User einloggen

# Grafiktreiber installieren:

72. `pacman -S nvidia oder nvidia-open nvidia-settings nvidia-utils`

# Display-Manager + DE:

74. `pacman -S xorg gdm gnome-control-center gnome-console nautilus xdg-user-dirs xdg-desktop-portal-gnome loupe file-roller ufw`

_sddm sddm-kcm plasma-desktop konsole --- kscreen plasma-nm plasma-pa pulseaudio-bluetooth powerdevil bluedevil khotkeys kinfocenter dolphin plasma-workspace-wallpapers kde-gtk-config_

_"pacman -S xorg lightdm lightdm-gtk-greeter xfce4 xdg-user-dirs pulseaudio pavucontrol"_

76. Gnome aktivieren: `systemctl enable gdm` / XFCE: `systemctl enable lightdm.service`

`ufw enable`
`systemctl enable firewalld.service`
`systemctl enable ufw.service`

`sudo ufw status verbose`

78. `reboot`

You can also set such limit as permanent and never worry about cleaning the logs. Just edit the file `/etc/systemd/journald.conf` by uncommenting `SystemMaxUse=` and setting the size limit: **`SystemMaxUse=50M`**

## NVIDIA Settings

für das cogwheel in GDM für wayland oder xorg, 
`nvidia-drm.modeset=1` in den kernel parametern eingeben:
`sudo nano /etc/default/grub` → linux default hinter quiet den oberen parameter eingeben.
danach `grub-mkconfig -o /boot/grub/grub.cfg` starten

To force-enable Wayland, override these rules by creating the following symlink:

- `ln -s /dev/null /etc/udev/rules.d/61-gdm.rules`

### screenshare usw. unter waylaynd

You need to install both `gst-plugin-pipewire` and `gst-plugins-good`. They are marked as optional dependencies of gnome-shell for screen recording.

**_für Firefox wayland das bild_** --> braucht man nicht mehr da Firefox automatisch erkennt ob wayland läuft

![](https://lh7-us.googleusercontent.com/JPLayvMi3ZiLVVBvi7TGvlez8h4x2q3nuSE-R7FsjtFp05Z4N6Gke2dySqj0f5Wb9Bb8yn8b7E2vB83mKuubrsm9-DXNmF7JruOK2p-U5Auh_qhLre6iHilYv3YqjTSoeQW9hENsdapRAYRCEnXGCrw)
