# MyLinux-Gentoo
Guide de création et configuration d'une machine Linux (e.g Gentoo)

## **Gentoo Setup Guide on VMware**

### **Introduction**
Ce guide décrit les étapes nécessaires pour installer et configurer Gentoo dans une machine virtuelle (VM) en utilisant VMware Workstation. Il est basé sur le **Gentoo Handbook (AMD64)** et est conçu pour ceux qui souhaitent construire leur propre système Linux minimal à partir de la distribution Gentoo.

### **Prérequis**
Avant de commencer, assurez-vous d'avoir les éléments suivants :
- **VMware Workstation** (version 16.x ou plus récente)
- **Gentoo distribution** (AMD64, avec systemd) :
  - Téléchargez l'**ISO minimal d'installation** [ici](https://www.gentoo.org/downloads/).
  - Téléchargez l'**archive Stage 3 systemd** (amd64) [ici](https://www.gentoo.org/downloads/).
  
### **Étape 1 : Création de la VM dans VMware**
1. **Lancer VMware Workstation**.
2. Créer une nouvelle machine virtuelle (VM) :
   - Sélectionnez **Custom (advanced)** pour la configuration de la VM.
   - **Compatibilité** : Sélectionnez la version appropriée de VMware Workstation.
   - **Installer l'OS plus tard** : Sélectionnez cette option car nous utiliserons un ISO pour l'installation.
   - **Système d'exploitation invité** : Sélectionnez **Linux** et choisissez **Linux 6.x kernel 64-bit**.
   - **Nommer la VM** : Choisissez un nom pour votre VM (par exemple : "Gentoo_VM").
   - **Configuration des processeurs** : Sélectionnez **2 processeurs** avec **1 cœur par processeur**.
   - **Mémoire (RAM)** : Attribuez **4 Go de RAM**.
   - **Réglages du réseau** : Sélectionnez **NAT**.
   - **Contrôleur SCSI** : Sélectionnez **LSI Logic**.
   - **Disque dur** : Créez un nouveau disque dur virtuel et **séparez-le en plusieurs fichiers** pour faciliter la gestion.
   - Cliquez sur **Customize Hardware** pour configurer certains paramètres avant de terminer :
     - **CD/DVD (ISO)** : Sélectionnez le fichier ISO minimal de Gentoo que vous avez téléchargé et assurez-vous de cocher **Connect at power on**.
   
3. **Passer en mode UEFI** :
   - Allez dans **VM (barre d'outils)** > **Settings** > **Options** > **Advanced** et sélectionnez **UEFI** au lieu de BIOS.

Votre VM est prête à être lancée !

### **Étape 2 : Installation de Gentoo**
Suivez les instructions dans le fichier `instructions.md` pour configurer un environnement minimal basé sur Gentoo, qui servira de base à la création de votre propre système d'exploitation.

### **Étape 3 : Compilation et configuration du noyau**
Une fois que l'installation de base est terminée, vous pouvez suivre le fichier `mykernel.md` pour commencer à configurer et compiler votre propre noyau Linux à partir des sources. Ce processus vous permettra d'adapter le noyau à vos besoins spécifiques.

### **Documentation Complémentaire**
- [Gentoo Handbook - AMD64](https://wiki.gentoo.org/wiki/Handbook:AMD64) : La documentation officielle de Gentoo qui est la principale référence pour l'installation et la configuration de Gentoo.
- [VMware Workstation User Guide](https://www.vmware.com/support/pubs/ws_pubs.html) : Guide officiel de VMware pour toute assistance sur les fonctionnalités de VMware.

### **Conclusion**
Ce dépôt est une documentation complète pour la mise en place d'un système Gentoo minimal dans une machine virtuelle. L'objectif est d'apprendre à construire un système d'exploitation à partir de zéro en partant des sources, ce qui permet d'acquérir une compréhension approfondie des composants d'un OS Linux.

---
