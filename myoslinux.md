
## Étape 2 : Création d’un OS Linux à partir des sources sur /dev/sda5

### Introduction

Dans cette étape, nous allons créer un système Linux complet sans utiliser de gestionnaire de paquets, mais directement en téléchargeant et en compilant les sources. Nous allons configurer et assembler le système sur la partition `/dev/sda5`.

### Préparation

Avant de commencer, nous devons monter les partitions nécessaires :

```bash
mount /dev/sda1 /efi
mount /dev/sda2 /boot
```

### I. Téléchargement des sources

Nous devons d'abord télécharger les fichiers sources nécessaires dans le répertoire `/usr/src`, monté sur `/dev/sda4`.

1. **Glibc** : Utilisez la même version que celle de votre machine Gentoo (vérifiez avec `ldd --version`).

   ```bash
   wget http://ftp.gnu.org/gnu/libc/glibc-2.39.tar.gz -P /usr/src
   ```

2. **Kernel** : Téléchargez la version indiquée pendant le cours théorique, par exemple la 6.6.54.

   ```bash
   wget https://mirrors.edge.kernel.org/pub/linux/kernel/v6.x/linux-6.6.54.tar.gz -P /usr/src
   ```

3. **Busybox** :

   ```bash
   wget https://busybox.net/downloads/busybox-1.32.0.tar.bz2 -P /usr/src
   ```

### II. Création du FHS (Filesystem Hierarchy Standard)

Nous allons créer le système de fichiers sur `/dev/sda5`, puis monter cette partition et y créer la hiérarchie de répertoires nécessaire.

1. Créer le système de fichiers ext4 sur `/dev/sda5` et monter la partition :

   ```bash
   mkfs.ext4 /dev/sda5
   mkdir -p /mnt/monlinux
   mount /dev/sda5 /mnt/monlinux
   ```

2. Créer la hiérarchie des répertoires :

   ```bash
   mkdir -p /mnt/monlinux/{bin,boot,dev,etc,home,lib64,mnt,proc,root,sbin,sys,tmp,usr,var}
   mkdir -p /mnt/monlinux/usr/{bin,include,lib64,local,sbin,src,share}
   mkdir -p /mnt/monlinux/usr/share/man/{man1,man2,man3,man4,man5,man6,man7,man8}
   mkdir -p /mnt/monlinux/var/{lock,log,run,spool}
   ln -s /usr/share/man /mnt/monlinux/usr/man
   ```

### III. Compilation et installation de Glibc

1. Extraire les fichiers Glibc :

   ```bash
   tar -xzf /usr/src/glibc-2.39.tar.gz -C /usr/src
   ```

2. Créer un répertoire pour compiler Glibc :

   ```bash
   mkdir /usr/src/glibc-build
   cd /usr/src/glibc-build
   ```

3. Configurer et compiler :

   ```bash
   CFLAGS="-O2 -U_FORTIFY_SOURCE -march=x86-64 -pipe" ../glibc-2.39/configure --enable-addons --prefix=/usr --with-headers=/usr/include
   make -j2
   make install_root=/mnt/monlinux install
   ```
   debut de la compilation glibc : 18:13:44
   fin de la compilation glibc : 19:31:16
### IV. Compilation et installation de Busybox

1. Décompresser Busybox :

   ```bash
   tar -xvjf /usr/src/busybox-1.32.0.tar.bz2 -C /usr/src
   ```

2. Configurer et compiler :

   ```bash
   cd /usr/src/busybox-1.32.0
   make menuconfig
   make -j2
   make CONFIG_PREFIX=/mnt/monlinux install
   ```
   debut de la compilation busybox : 20:14:30
   fin de la compilation busybox : 20:16:13
### V. Création des fichiers de configuration

1. Créer le fichier `passwd` :

   ```bash
   echo "root::0:0:root:/root:/bin/ash" > /mnt/monlinux/etc/passwd
   ```

2. Créer le fichier `fstab` :

   ```bash
   echo "/proc /proc proc defaults" > /mnt/monlinux/etc/fstab
   ```

3. Créer le fichier de mappage clavier `monclavier.kmap` :

   ```bash
   /mnt/monlinux/bin/busybox dumpkmap > /mnt/monlinux/etc/monclavier.kmap
   ```

4. Créer le script de démarrage `rc` :

   ```bash
   nano /mnt/monlinux/etc/rc
   ```

   Contenu du fichier `rc` :

   ```bash
   #!/bin/ash
   /bin/mount -av
   /bin/hostname NOMDEFAMILLE
   /sbin/loadkmap < /etc/monclavier.kmap
   ```

   Donner les droits d'exécution :

   ```bash
   chmod +x /mnt/monlinux/etc/rc
   ```

5. Créer le fichier `inittab` :

   ```bash
   nano /mnt/monlinux/etc/inittab
   ```

   Contenu du fichier `inittab` :

   ```bash
   ::sysinit:/etc/rc
   tty1::respawn:/bin/ash -l
   ```

### VI. Compilation du noyau

1. Extraire les sources du noyau :

   ```bash
   tar -xzf /usr/src/linux-6.6.54.tar.gz -C /usr/src
   cd /usr/src/linux-6.6.54
   ```

2. Générer un fichier de configuration par défaut :

   ```bash
   make defconfig
   ```

3. Configurer le noyau avec `menuconfig`, puis compiler :

   ```bash
   make menuconfig #Faire la même config que dans gentoo voir instructions.md
   make -j2
   ```
   debut de la compilation kernel : 21:19:46
   fin de la compilation kernel : 22:00:45
4. Copier le noyau dans la partition de démarrage :

   ```bash
   cp /usr/src/linux-6.6.54/arch/x86/boot/bzImage /boot/kernel-linux-6.6.54-monlinux
   ```

### VII. Configuration de GRUB

1. Mettre à jour `grub.cfg` :

   ```bash
   grub-mkconfig -o /boot/grub/grub.cfg
   ```

2. Modifier le fichier pour démarrer sur `/dev/sda5` :

   ```bash
   nano /boot/grub/grub.cfg
   ```

   Exemple d'entrée :

   ```bash
   linux /boot/kernel-linux-6.6.54-<hostname> root=/dev/sda5 console=tty0 console=ttyS1 rootfstype=ext4 ro
   ```

---

### Configuration de la console TTY1 et finalisation

Pour activer la console **TTY1** et capturer les sorties de démarrage :

1. Éteindre la VM.
2. Aller dans les paramètres de la VM : **Settings > Serial port > Use output file** (spécifiez un chemin sur votre machine physique).

Cela permet de capturer les messages de la console, utiles pour diagnostiquer des erreurs de démarrage.

Une fois configuré, vous pouvez redémarrer la VM et utiliser votre nouveau noyau. Si des problèmes apparaissent (par exemple, pas de `hostname` ou mauvais clavier), vérifiez votre script de démarrage `/etc/rc`

---
---
