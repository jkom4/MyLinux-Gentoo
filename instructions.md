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
   - `/dev/sda1` : 100M EFI system (100 Mo pour UEFI).
   - `/dev/sda2` : 200M Linux filesystem (boot).
   - `/dev/sda3` : 2G Linux swap (mémoire de swap).
   - `/dev/sda4` : 20G Linux filesystem (partition pour le système).
   - `/dev/sda5` : 2.7G Linux filesystem

---

## **3. Création du système de fichiers et montage des partitions**

1. **Créer les systèmes de fichiers :**
   ```bash
   mkfs.vfat -F32 /dev/sda1  # UEFI
   mkfs.vfat -F32 /dev/sda2  # Boot
   mkswap /dev/sda3  # Swap
   mkfs.ext4 -T small /dev/sda4  # Partition système (-T small veut dire qu'il va prévoir un nombre de fichiers très grand car on utilisera beaucoup de petit fichier sinon à la base il calcul le nombre de fichier logique pour la taille de la partition)
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
4. ** Recharger avec le nouvel environnement : **
    ```bash
   env-update && source /etc/profile && export PS1="(chroot) ${PS1}"
   ```
---

## **7. Compilation et installation du noyau**

1. **Configuration matérielle et outils de base :**
```bash
emerge --ask sys-apps/pciutils #Installe l'outil lspci, qui permet d'inspecter le matériel PCI (cartes graphiques, cartes réseau, etc.).
 ```
```bash
emerge --ask sys-apps/usbutils # Installe l'outil lsusb, pour détecter et afficher les périphériques connectés aux ports USB.
 ```
```bash
lspci -k # Affiche les périphériques PCI et les pilotes associés actuellement chargés.
 ```
```bash
lsusb # Liste les périphériques USB connectés.
 ```
```bash
lsmod # Affiche les modules (pilotes) actuellement chargés dans le noyau.
 ```

2. **Télécharger les sources du noyau :**
   ```bash
   emerge --ask sys-kernel/gentoo-sources
   cd /usr/src/linux
   make menuconfig  # Configurer les options du noyau
   ```
---

# Kernel Configuration for Gentoo Linux

Below is the complete list of required kernel configuration options. Use `make menuconfig` to access the configuration interface and apply these settings:

```bash
# File Systems
File systems → FUSE (Filesystem in Userspace) support
File systems → DOS/FAT/EXFAT/NT Filesystems → NTFS filesystem support
File systems → DOS/FAT/EXFAT/NT Filesystems → NTFS read-write filesystem support
File systems → Pseudo filesystems → EFI Variable filesystem

# Partition Types
Enable the block layer → Partition Types → Advanced partition selection

# Firmware Drivers
Device Drivers → Firmware Drivers → EFI → EFI Bootloader Control
Device Drivers → Firmware Drivers → EFI → EFI capsule loader
Device Drivers → Firmware Drivers → EFI → EFI Runtime Service Tests support

# Fusion MPT Device Support
Device Drivers → Fusion MPT device support
Device Drivers → Fusion MPT device support → Fusion MPT ScsiHost drivers for SPI

# Graphics Support
Device Drivers → Graphics support → Frame buffer Devices → Support for frame buffer device drivers
Device Drivers → Graphics support → Frame buffer Devices → EFI-based Framebuffer Support
Device Drivers → Graphics support → Console display driver support → Framebuffer Console support
# SCSI Device Support
Device Drivers → SCSI device support → SCSI low-level drivers → VMware PVSCSI driver support

# Miscellaneous Drivers
Device Drivers → Misc devices → VMware VMCI Driver

# Init Systems
Gentoo Linux → Support for init systems, system, and service managers → systemd
```

À la fin de la configuration, il est important de vérifier si le support pour "FUSION MPT" est activé :

```bash
cat .config | grep FUSION # Vérifiez si 'CONFIG_FUSION=y' est présent
cat .config | grep CONFIG_FRAMEBUFFER_CONSOLE # Vérifiez si le framebuffer console est activé
```

Nous sommes maintenant prêts à compiler le noyau. Il est conseillé de noter l'heure avant et après la compilation pour calculer le temps total nécessaire.

> **Note :** Cette compilation va générer des fichiers objets qui occuperont temporairement plus de 10 Go d'espace disque. À la fin, le fichier exécutable du noyau (binaire compacté) ne fera qu'environ 10 Mo.

--- 


3. **Compiler et installer le noyau :**
   ```bash
   make -j2
   make install
   make modules_install
   ```
   Debut compilation 14:19:24  Fin : 14:37:19
---

Voici une version traduite et reformulée en français pour intégrer dans votre README :

---

# Étapes après la compilation du noyau

Le fichier exécutable du noyau est généré dans :  
`/usr/src/linux-6.6.47-gentoo/arch/x86/boot/bzImage`
1. Installez les utilitaires nécessaires :  
   ```bash
   emerge sys-kernel/installkernel
   emerge dracut
   ```
2. Modifiez le fichier de configuration pour l'installation du noyau :  
   ```bash
   nano /usr/lib/kernel/install.conf
   ```
   - Modifiez :  
     ```
     layout=grub
     initrd_generator=dracut
     ```
     - Ne modifiez pas les autres options.

3. Modifiez le fichier `fstab` :  
   ```bash
   nano /etc/fstab
   ```
   - Exemple de configuration :  
     ```
     /dev/sda4  /      ext4   defaults,noatime  0 1
     /dev/sda3  none   swap   sw                0 0
     ```

---

### 3. Donner un nom à votre système
```bash
echo NOMFAMILLE > /etc/hostname
```

---

### 4. Configurer un client DHCP
1. Installez le client DHCP :  
   ```bash
   emerge dhcpcd
   ```
2. Configurez le mot de passe root :  
   ```bash
   passwd
   ```

---

### 5. Configurer le chargeur de démarrage GRUB
1. Créez le répertoire EFI :  
   ```bash
   mkdir -p /efi/EFI/Gentoo
   ```

2. Ajoutez la prise en charge EFI dans les options de compilation :  
   ```bash
   echo 'GRUB_PLATFORMS="efi-64"' >> /etc/portage/make.conf
   emerge sys-boot/grub
   ```

3. Installez GRUB :  
   ```bash
   grub-install --target=x86_64-efi --efi-directory=/efi --removable
   ```

4. Vérifiez la présence des fichiers :  
   ```bash
   cd /efi/EFI/BOOT
   cd /boot/grub
   ```

5. Copiez le noyau généré :  
   ```bash
   cp /usr/src/linux-6.6.47-gentoo/arch/x86/boot/bzImage /boot/kernel-6.6.54-NOMFAMILLE
   uname -a
   ```

6. Générer la configuration de GRUB :  
   ```bash
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

---

### 6. Faire une sauvegarde avant de redémarrer
Prenez un snapshot ou sauvegardez votre configuration actuelle pour éviter toute perte de données.

---

### 7. Redémarrer le système
1. Quittez l'environnement de chroot :  
   ```bash
   exit
   cd
   umount -l /mnt/gentoo/dev{/shm,/pts,}
   umount -lR /mnt/gentoo
   reboot
   ```

---

### 8. Configurations post-démarrage
1. Connectez-vous avec les identifiants suivants :  
   - **Utilisateur** : `root`  
   - **Mot de passe** : `YOURPASSWORD2024`  
     > ⚠️ Le clavier est en **QWERTY** par défaut.

2. Supprimez la configuration de la console virtuelle :  
   ```bash
   rm /etc/vconsole.conf
   ```

3. Configurez le clavier :  
   ```bash
   systemd-firstboot --prompt-keymap
   ```

4. Activer DHCP :  
   ```bash
   systemctl enable dhcpcd
   ```

5. Démarrer le serveur SSH :  
   ```bash
   systemctl start sshd
   systemctl enable sshd
   ```

Votre système est maintenant prêt à être utilisé.

---
