Voici le texte que tu peux copier directement dans un fichier `.md` pour GitHub :

---

# **Installation d'une machine Gentoo**

## **1. Configuration initiale et SSH**

1. **Changer la disposition du clavier :**
   ```bash
   loadkeys be-latin1  # Pour passer en disposition de clavier belge
   ```
2. **Définir le mot de passe root :**
   ```bash
   passwd  # Définir un mot de passe pour root
   ```
3. **Créer un utilisateur :**
   ```bash
   useradd -m -G users,wheel user  # Créer un utilisateur avec accès aux groupes users et wheel
   passwd user  # Définir un mot de passe pour l'utilisateur
   ```
4. **Démarrer le service SSH :**
   ```bash
   /etc/init.d/sshd start  # Démarrer SSH
   ```
5. **Vérifier l'adresse IP de la VM :**
   ```bash
   ip addr  # Affiche l'adresse IP de la VM
   ```

### **Connexion SSH depuis Windows :**
- Utilisez `ssh user@<IP>` depuis votre terminal Windows pour vous connecter à la VM via SSH.

---

## **2. Partitionnement du disque**

1. **Passer en root :**
   ```bash
   su -  # Passer en mode super-utilisateur
   ```
2. **Configurer les partitions :**
   ```bash
   fdisk /dev/sda  # Configurer les partitions
   ```
   - **g** : Modifier le disque pour utiliser GPT (au lieu de MBR).
   - **n** : Créer une nouvelle partition (ex. 100 Mo pour /boot).
   - **p** : Afficher les informations sur le disque et les partitions.
   - **t** : Changer le type de partition (ex. UEFI).
   - **w** : Enregistrer les modifications.

   **Exemple de partitions créées :**
   - `/dev/sda1` : FAT32 (100 Mo pour UEFI).
   - `/dev/sda2` : FAT32 (boot).
   - `/dev/sda3` : Swap (mémoire de swap).
   - `/dev/sda4` : ext4 (partition pour le système).

---

## **3. Création du système de fichiers et montage des partitions**

1. **Créer les systèmes de fichiers :**
   ```bash
   mkfs.vfat -F32 /dev/sda1  # UEFI
   mkfs.vfat -F32 /dev/sda2  # Boot
   mkswap /dev/sda3  # Swap
   mkfs.ext4 -T small /dev/sda4  # Partition système
   ```
2. **Monter les partitions :**
   ```bash
   mount /dev/sda4 /mnt/gentoo  # Monter la partition système
   mkdir /mnt/gentoo/boot
   mount /dev/sda2 /mnt/gentoo/boot  # Monter la partition boot
   mkdir /mnt/gentoo/efi
   mount /dev/sda1 /mnt/gentoo/efi  # Monter la partition UEFI
   swapon /dev/sda3  # Activer la partition de swap
   ```

---

## **4. Installation du stage3**

1. **Copier le fichier Stage3 sur la VM :**
   Depuis une autre machine, copiez le fichier Stage3 sur la VM :
   ```bash
   scp <stage3filename.tar.xz> user@<IP_VM>:/home/user
   ```
2. **Déplacer et extraire l'archive :**
   ```bash
   mv /home/user/<stage3filename> /mnt/gentoo
   cd /mnt/gentoo
   tar xpvf stage3-*.tar.xz --xattrs-include='*.*' --numeric-owner
   rm <stage3filename>  # Supprimer l'archive après extraction
   ```

---

## **5. Montage des systèmes virtuels et chroot**

1. **Monter les systèmes :**
   ```bash
   mount --types proc /proc /mnt/gentoo/proc
   mount --rbind /sys /mnt/gentoo/sys
   mount --make-rslave /mnt/gentoo/sys
   mount --rbind /dev /mnt/gentoo/dev
   mount --make-rslave /mnt/gentoo/dev
   mount --bind /run /mnt/gentoo/run
   mount --make-slave /mnt/gentoo/run
   ```

2. **Entrer dans l'environnement chroot :**
   ```bash
   chroot /mnt/gentoo /bin/bash
   source /etc/profile
   export PS1="(chroot) ${PS1}"
   ```

---

## **6. Configuration du système**

1. **Synchroniser le dépôt :**
   ```bash
   emerge-webrsync
   ```
2. **Sélectionner le profil systemd :**
   ```bash
   eselect profile list
   eselect profile set 22  # Choisir le profil systemd
   ```

3. **Configurer le fuseau horaire :**
   ```bash
   ln -sf /usr/share/zoneinfo/Europe/Brussels /etc/localtime
   ```
4. **Configurer la langue et le clavier :**
   ```bash
   nano /etc/locale.gen
   # Décommentez 'fr_BE.UTF-8 UTF-8'
   locale-gen
   eselect locale list
   eselect locale set 5  # Sélectionner la langue
   ```

---

## **7. Compilation et installation du noyau**

1. **Télécharger les sources du noyau :**
   ```bash
   emerge --ask sys-kernel/gentoo-sources
   cd /usr/src/linux
   make menuconfig  # Configurer les options du noyau
   ```
liste des configurations :
    Gentoo Linux > Support > support for init > systemd
    Device drivers > Generic driver options > Maintain a devtmpfs filesystem to mount at /dev
    skip NMVE (we don't have it)
    Enable the block layer > partition type > advanced > EFI GUID partition support
    File system > DOS/FAT/MSDOS > check the 2 NTFS
    Device drivers > Firmware drivers > EFI > check the following... - EFI Runtime config interface table version 2 - EFI bootloader control - EFI capsule loader - EFI Runtime service tests support
    Device drivers > Fusion MPT > check Fustion MPT SCI Host drivers for SPI
    Device drivers > Graphic support > Framebuffer devices > support for frame buffer device > EFI based frame buffer support (enable it)
    Device drivers > SCSI device support > SCSI low-level drivers > VMware PVSCSI driver support
    Device drivers > MISC devices > VMware MCI driver
    Processor type and features > EFI runtime service support > EFI stub support > disable (space)
    Device Drivers > Graphics support > Console display driver support > Framebuffer Console support (enable)
    File systems > FUSE (Filesystem in Userspace) support
    File systems > Pseudo filesystems > EFI Variable filesystem
2. **Compiler et installer le noyau :**
   ```bash
   make -j2
   make install
   make modules_install
   ```
---

## **8. Installation du bootloader (GRUB)**

1. **Installer et configurer GRUB :**
   ```bash
   echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
   emerge sys-boot/grub
   grub-install --target=x86_64-efi --efi-directory=/efi
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

2. **Créer un nom pour le système :**
   ```bash
   echo 'mon_gentoo' > /etc/hostname
   ```

---

## **9. Finalisation et démarrage du SSH**

1. **Créer un snapshot de la VM avant de redémarrer :**
   ```bash
   exit
   umount -l /mnt/gentoo/dev{/shm,/pts}
   umount -R /mnt/gentoo
   reboot
   ```

2. **Activer le DHCP et SSH au redémarrage :**
   ```bash
   rm /etc/vconsole.conf   
   systemd-firstboot --prompt-keymap  #Pour configurer le clavier 
   systemctl enable dhcpcd
   systemctl start sshd
   systemctl enable ssh
   dhcpcd  #pour configurer le service
   ```



---
