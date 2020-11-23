# Administration des systèmes Linux TD3 – LXC/Docker

## 2-Désactivez SELinux
```bash
setenforce 0
```
## 3-Désactivez le pare-feu.
```bash
systemctl stop firewalld
```
## 4- Origine des conteneurs
### [4A] Qu’est ce qu’une « jails » ?
C'est une architecture logicielle du système d'exploitation FreeBSD, qui permet de compartimenter des processus
### [4B] Dans quels cadres utilise-t-on les « jails »?
La technologie jail permet non seulement d'exécuter des conteneurs, mais aussi de simplifier leur conception et leur fabrication, l'envoi d'images, le contrôle des versions d'image, etc
### [4C] Quel est l’équivalent des « jails » sous Linux ?
L'équivalent de jails sous linux est **LXC**
### [4D] Qu’est ce qu’un CGROUP sous Linux ?
Groupe d'user

## 5- Les conteneurs

### [5A] Quel est le principe de la conteneurisation ?
La conteneurisation, à ne pas confondre avec la virtualisation est une méthode permettant d’exécuter une application dans un environnement virtuel dans une seule zone appelée Conteneur. Seul l’environnement d’exécution est virtualise dans un conteneur (systèmes de fichiers, réseau, processeur, mémoire vive, …). L’application exécuter dans cet environnement virtuel stocke tous les fichiers, bibliothèque et librairies dans le conteneur.
Ensuite notre conteneur se connecte au noyau (kernel) d’un système d’exploitation. Ici le noyau gèrent les ressources de l’ordinateur et permet au différents composants matériels et logiciels de communiquer entre eux. La conteneurisation rend donc le déplacement d’application virtuelle plus simple entre des systèmes d’exploitation identiques et demande moins de ressources de mémoire, de RAM, de CPU, etc.d
A la base, le concept de conteneurisation permet aux instances virtuelles de partager un système d'exploitation hôte unique, avec ses fichiers binaires, bibliothèques ou pilotes.
On utilise donc le même système d'exploitation (OS) hôte pour plusieurs conteneurs, au lieu d'installer un OS (et d'en acheter la licence) pour chaque VM invitée
### [5B] Quelle est la différence essentielle entre un conteneur et une VM ?
Il n’est donc pas nécessaire comme pour les machines virtuelle, d’installer un nouveau système d’exploitation
### [5C] Quels sont les différents type de conteneur existant ?

## 6- Installation et prise en mains de LXC
### [6A] Activez et démarrez le service libvirtd. Quel est son rôle ?
systemctl status libvirtd
La bibliothèque libvirt est utilisée pour s'interfacer avec différentes technologies de virtualisation.
### [6B] Activez et démarrez le service lxc. Quel est son rôle ?
LXC est une interface d'espace utilisateur pour les fonctionnalités de confinement du noyau Linux. Grâce à une API puissante et à des outils simples, il permet aux utilisateurs de Linux de créer et de gérer facilement des conteneurs système ou d'application.

## 7- Création d’un conteneur LXC
### [7A] Il existe 2 façons rapides de créer localement un conteneur : Quelles sont elles ?
nteractively building Images
Using Dockerfile
Importing from a tarball
### [7B] Dans quelles conditions pouvez vous rencontrer ce message d’erreurs ?
### [7C] Qu’est ce qu’un serveur de clefs ? Sur quel port écoute-t-il ?
### [7D] Quelles solutions proposez vous ?
### [7E] Quelle est la commande permettant de déployer un conteneur LXC de type debian ?
### [7F] Déployez un conteneur centos, version 7. Quelle commande avez vous utilisée ? A
quoi sert l’option -n ?
### [7G] Démarrer le conteneur. Quelle commande avez-vous utilisée ? Quelle option vous
permet de démarrer le conteneur en background ?
### [7H] A quoi servent les commandes
-lxc-info ?
-lxc-execute ?
-lxc-attach ?
-lxc-top ? Quel est l’empreinte memoire de votre conteneur ?
### [7I] A quoi sert l’option -B de la commande lxc-create ?
### [7J] Que contient le fichier /etc/lxc/default.conf ?
### [7K] Ou est déployer le conteneur créé ?
### [7L] Que contient le répertoire « rootfs » de ce répertoire ?
### [7M] Que contient le fichier config de ce répertoire ?
### [7N] Dans la VM, la commande ip addr fait apparaître un périphérique nommé virbr0.
Qu’est ce que ce périphérique. A quoi sert-il ?
### [7O] Si votre conteneur LXC est bien démarré, la commande ip addr doit aussi afficher
un périphérique dont le nom commence par veth. Qu’est ce que ce périphérique ? Quel est son
rôle ?
### [7P] Lors de la création du conteneur, aucun mdp ne vous a été demandé. Proposez une
solution pour changer le mot de passe
### [7Q] Vous avez maintenant 2 options pour vous connecter sur le conteneur. Quels sont
elles ?
### [7R] « Loggez » vous sur le conteneur et :
- Donnez l’IP
- Installez le service ssh, démarrez-le. Quelles commandes avez vous utilisées ?
- Connectez-vous sur le conteneur en ssh depuis un autre terminal de votre VM
### [7S] Pouvez vous joindre le conteneur depuis Windows ? Si non pourquoi ?
### [7T] Imaginez que vous deviez déployer très souvent des conteneurs avec des services
Apache et mySQL. Quels sont les problèmes que vous allez rencontrer ? Quels solutions proposez
vous ?
