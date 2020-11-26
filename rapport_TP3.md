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
CGroups d'user est une fonctionnalité du noyau linux permettant de contrôler et limiter l’utilisation des ressources pour un utilisateur ou un groupe d’utilisateurs

## 5- Les conteneurs
### [5A] Quel est le principe de la conteneurisation ?
La conteneurisation permet d’isoler une application particulière dans une image. Une image peut être utilisé comme modèle afin de créer un ou plusieurs conteneurs. L'avantage principale est d'utiliser qu’un seul système d’exploitation, avec une couche docker ou LXC gérant les instances d’images
### [5B] Quelle est la différence essentielle entre un conteneur et une VM ?
Il n’est donc pas nécessaire comme pour les machines virtuelle, d’installer un nouveau système d’exploitation
### [5C] Quels sont les différents type de conteneur existant ?

## 6- Installation et prise en mains de LXC
### [6A] Activez et démarrez le service libvirtd. Quel est son rôle ?
Elle est utilisée pour s'interfacer avec différentes technologies de virtualisation
```bash
systemctl start libvirtd
systemctl status libvirtd
```
### [6B] Activez et démarrez le service lxc. Quel est son rôle ?
LXC est une interface d'espace utilisateur pour les fonctionnalités de confinement du noyau Linux qui permet aux utilisateurs de créer et de gérer facilement des conteneurs système ou d'application
```bash
systemctl start lxc
systemctl status lxc
```

## 7- Création d’un conteneur LXC
### [7A] Il existe 2 façons rapides de créer localement un conteneur : Quelles sont elles ?
```bash
lxc-create -t download -n nom_conteneur -- -d distribution -r release -a architecture
```
### [7B] Dans quelles conditions pouvez vous rencontrer ce message d’erreurs ?
C'est sûrement un problème de firewall ou proxy
### [7C] Qu’est ce qu’un serveur de clefs ? Sur quel port écoute-t-il ?
En sécurité informatique, un serveur de clés est un serveur qui :  
- reçoit des clés de chiffrement de leurs propriétaires  
- emmagasine ces clés  
- les distribue aux utilisateurs ou programmes qui veulent communiquer de façon chiffrée avec les propriétaires des clés

Et le serveur écoute sur le port 11371
### [7D] Quelles solutions proposez vous ?
On peut autoriser le port 11371 ou bien désactiver temporairement son firewwall
### [7E] Quelle est la commande permettant de déployer un conteneur LXC de type debian ?
```bash
lxc-create -t download -n nom_conteneur -- -d debian
```
### [7F] Déployez un conteneur centos, version 7. Quelle commande avez vous utilisée ? A quoi sert l’option -n ?
-n sert à nommer le conteneur
```bash
lxc-create -t download -n nom_conteneur -- -d centos
```
### [7G] Démarrer le conteneur. Quelle commande avez-vous utilisée ? Quelle option vous permet de démarrer le conteneur en background ?
option -d permet de démarrer le conteneur en background
```bash
lxc-start nom_conteneur
```
### [7H] A quoi servent les commandes
-lxc-info ? donne des infos sur le conteneur avec l’option -n  
-lxc-execute ? exécuter une application dans le containeur  
-lxc-attach ? permet de créer un shell s’éxecutant dans un conteneur  
-lxc-top ? Affiche les statistiques du conteneur  
-Quel est l’empreinte memoire de votre conteneur ? L’emprunte mémoire est l’utilisation de la mémoire d’un conteneur
### [7I] A quoi sert l’option -B de la commande lxc-create ?
Permet de spécifier l'option --dir ROOTFS , ce qui signifie que les rootfs du conteneur doivent être placés sous le chemin
### [7J] Que contient le fichier /etc/lxc/default.conf ?
Le fichier default.conf défini les paramètres de bases de lxc
### [7K] Ou est déployer le conteneur créé ?
**/var/lib/lxc**
### [7L] Que contient le répertoire « rootfs » de ce répertoire ?
### [7M] Que contient le fichier config de ce répertoire ?
La configuration du container
### [7N] Dans la VM, la commande ip addr fait apparaître un périphérique nommé virbr0. Qu’est ce que ce périphérique. A quoi sert-il ?
C'est l'interface utilisée pour la connexion en NAT entre machine et conteneur
### [7O] Si votre conteneur LXC est bien démarré, la commande ip addr doit aussi afficher un périphérique dont le nom commence par veth. Qu’est ce que ce périphérique ? Quel est son rôle ?
C'est une carte réseau virtuel qui permet au conteneur de se connecter à internet
### [7P] Lors de la création du conteneur, aucun mdp ne vous a été demandé. Proposez une solution pour changer le mot de passe
```bash
lxc-attach -n nom_conteneur
passwd
```
### [7Q] Vous avez maintenant 2 options pour vous connecter sur le conteneur. Quels sont elles ?
lxc-attach ou connection ssh
### [7R] « Loggez » vous sur le conteneur et :
- Donnez l’IP 192.168.122.174  
- Installez le service ssh, démarrez-le. Quelles commandes avez vous utilisées ?
```bash
lxc-attach -n nomConteneur
yum -y install openssh-server openssh-clients
systemctl enable sshd
nano /etc/ssh/sshd_config #enleve le commentaire de la ligne "PermitRootLogin yes" de la setcion #Authentification
```
- Connectez-vous sur le conteneur en ssh depuis un autre terminal de votre VM  
**ssh root@adresse_ip**  
### [7S] Pouvez vous joindre le conteneur depuis Windows ? Si non pourquoi ?
Impossible de joindre le conteneur depuis windows car on est pas sur le même réseau
### [7T] Imaginez que vous deviez déployer très souvent des conteneurs avec des services Apache et mySQL. Quels sont les problèmes que vous allez rencontrer ? Quels solutions proposez vous ?

## 8- Création d’un conteneur Docker
### [8A] Il existe 2 solutions pour créé un conteneur docker. Quelles sont elles ?
On peut utiliser un DockerFile ou créer un conteneur a partir d’une image
### [8B] La commande ip addr affiche un périphérique nommé docker0 ? A quoi correspondt-il ?
docker0 est un pont Linux, Docker sélectionne un sous-réseau qui n'est pas utilisé et attribue une adresse IP

## 9- Utilisation d’un Dockerfile
### [9A] Quel est le contenu du fichier Dockerfile qui est proposé ?
C'est une configuration pour un serveur Apache
### [9B] A quoi sert la directive FROM ?
Permet de séléctionner la version du service Apache
### [9C] A quoi sert la directive COPY ? 
Permet de copier une configuration
### [9D] A quoi correspond ./public-html/
Contient tous les fichiers html du site
### [9E] A quoi correspond /usr/local/apache2/htdocs/
C'est le dossier où se situe les fichier de configuration d'apache
### [9F] Quelle est la commande utilisée pour créer l’image docker ? Créez une image nommée « apachy »
```bash
docker build -t apachy 
```
### [9G] Quelle commande permet de lister les images présentent dans le dépôt local (Repository)
```bash
docker image ls
```
### [9H] Que contient votre dépôt local ?
### [9I] Votre dépôt ne devrait contenir que 2 images. Quelle commande devez vous utiliser pour supprimer une image ?
```bash
docker rmi image
```
###  [9J] Démarrez votre conteneur nommé « apachy »
```bash
docker run -dit –name apachy -p 8080:80 apachy
```
A quoi servent les options :  
--name : nom du conteneur  
--d : exécute le conteneur en arrière-plan  
--it : Garde STDIN ouvert  
--p : Publie le port d'un conteneur sur l'hôte
### [9K] Pouvez vous vérifier que votre conteneur est lancé ? Comment ?
```bash
docker ps
```
### [9L] La commande ip addr affiche un périphérique particulier. Lequel ?
C'est une carte réseau virtuel 
### [9M] Quelle commande permet d’afficher l’IP du conteneur ?
```bash
docker inspect apachy
```
### [9N] Essayez de joindre le serveur web du conteneur, quelle réponse obtenez vous ?
On n'arrive pas à le joindre 
### [9O] Récupérez le fichier info.php qui doit être présent sur le serveur web du conteneur « apachy ». Qu’observez-vous ?
Le serveur ne gère pas le PHP

## 10- Création d’une image personnalisée avec un Dockerfile
### [10A] A quoi sert la commande RUN présente dans un fichier Dockerfile
Permet d'exécuter l'ordre donné
### [10A] Lancez l’image. Quelle démarche avez-vous utilisée pour vérifier :
-Que l’image est lancée ?
-Que le serveur web fonctionne ?
-Que le moteur php dans le conteneur répond correctement.
```bash
docker build -t Web .
docker run –name TestWeb -p 8080:80 Web
```
