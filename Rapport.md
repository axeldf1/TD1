# 2-Désactivation de SELinux
### [2A] A quoi sert ce composant ?
Security Enhanced Linux est un Module de sécurité Linux
### [2B] Comment peut on le désactiver momentanément (Il existe au moins 2 moyens)
```bash
setenforce 0
echo "0" > /selinux/enforce
```
### [2C] Comment peut-on le désactiver de façon permanente ?
Il faut éditer le fichier **/etc/selinux/config** en remplançant :
  - SELINUX=enforcing par SELINUX=permissive
  
# 3-Désactivation momentané du pare-feu.
### [3A] Quelle commande vous permet de désactiver temporairement le firewall, le temps de
procéder a la configuration de la machine ?
```bash
systemctl stop firewalld
```
### [3B] Quelle commande vous permet d’avoir la certitude que le firewall est bien inactif ?
```bash
systemctl is-enabled firewalld
systemctl start firewalld #pour le réactiver
```

# 4-Installation du service Apache (La configuration réseau avancé viendra par la suite)
### [4A] Quels sont les trois principaux serveurs WEB disponible sous Linux
Apache HTTP Server, Nginx et Apache Tomcat
### [4B] La commande dnf (ou anciennement yum) permet d’installer les packages.
-Quelle commande permet de retrouver le nom du package contenant le serveur
Apache ?
```bash
dnf list httpd
```
-Quelle commande permet d’installer le serveur Apache ?
```bash
dnf install httpd
```
### [4C] Démarrage du service Apache
-Quelle commande permet de démarrer le service Apache ?
```bash
systemctl start httpd
```
### [4D] Quelle commande vous permet d’avoir la certitude que le service Apache est
fonctionnel ?
```bash
systemctl status httpd
```
### [4E] Quelles informations essentielles vous sont fournies par cette commande ?

## 5-Configuration du service Apache
### [5A] Ou se trouvent les fichiers de configuration de Apache ?
**/etc/httpd**
### [5B] Quelle est la structure de ce répertoire ?
```bash
/etc/httpd
├── conf
│   ├── httpd.conf
│   └── magic
├── conf.d
│   ├── autoindex.conf
│   ├── README
│   ├── userdir.conf
│   └── welcome.conf
├── conf.modules.d
│   ├── 00-base.conf
│   ├── 00-dav.conf
│   ├── 00-lua.conf
│   ├── 00-mpm.conf
│   ├── 00-optional.conf
│   ├── 00-proxy.conf
│   ├── 00-systemd.conf
│   ├── 01-cgi.conf
│   ├── 10-h2.conf
│   ├── 10-proxy_h2.conf
│   └── README
├── logs -> ../../var/log/httpd
├── modules -> ../../usr/lib64/httpd/modules
├── run -> /run/httpd
└── state -> ../../var/lib/httpd
```
### [5C] Ou se trouve le fichier de configuration principal ?
**/etc/httpd/conf/httpd.conf**
### [5D] Que contient (du point de vue logique) le répertoire conf.d
Dossier contenant les fichiers secondaires de configuration, fournis par les extensions et les logiciels utilisant Apache
### [5E] Que contient le répertoire conf.modules.d
Dossier contenant les fichiers de configurations des extensions
### [5F] Qu’est-ce qu’un module ?
Un module est un morceau de code permettant d'ajouter des fonctionnalités au noyau, c'est une fonctionnalité
### [5G] Ou sont-ils stockés ?
**/etc/httpd/modules**
### [5H] Ou sont stockés les log de Apache ?
**/etc/httpd/logs**
##-Exploration de la configuration de base (fichier httpd.conf)
### [6A] Qu’est ce que le ‘DOCUMENT ROOT’ ?
C'est la racine principale de l'arborescence des documents visible depuis Internet
### [6B] Sur quel répertoire pointe-t-il ?
**/var/www/html*
### [6C] A quoi sert la directive ‘Listen’
Permet de paramétrer sur quel IP et/ou port Apache va écouter
### [6D] A quoi sert la déclaration <Directory >
Les balises <Directory> et </Directory> permettent de regrouper un ensemble de directives qui ne s'appliquent qu'au répertoire précisé
### [6E] A quoi servent les directives ‘AllowOverride’ et ‘Require’ ?
AllowOverride : Types de directives autorisées dans les fichiers .htaccess  
Require : Détermine les utilisateurs authentifiés autorisés à accéder à une ressource
### [6F] A quoi sert le fichier welcome.conf ?
C'est le fichier par défaut si l'index n'est pas indiqué
### [6G] A quoi sert la commande apachectl ?
Active les connexions HTTP persistantes

## 7-Tests et Installations complémentaires

### [7A] Exécutez la commande yum install php -y , Quels packages sont installés ?
```bash
php-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64    
php-fpm-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64   
nginx-filesystem-1:1.14.1-9.module_el8.0.0+184+e34fea82.noarch   
php-cli-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
php-common-7.2.24-1.module_el8.2.0+313+b04d0a66.x86_64
```
### [7B] Le service Apache utilise désormais systématiquement le package ‘php-fpm’ contenant les éléments nécessaires pour l’exécution de pages web dynamiques. Que contient ce package ?
```bash
rpm -qi php-fpm
```
Ce paquet contient l'interpréteur FastCGI qui s'exécute en tant que démon et reçoit les requêtes Fast / CGI
### [7C] Comment prouver que le serveur Web est désormais capable d’exécuter du PHP ?
```bash
systemctl status httpd

Drop-In: /usr/lib/systemd/system/httpd.service.d
           └─php-fpm.conf
```
### [7D] Quelle version de php est installée ?
```bash
php -v
```
on utilise la version : PHP 7.2.24
### [7E] Comment connaître la configuration compléte du serveur web (librairies installées, avec leurs versions et configurations?)
```bash
php -info
```
### [7F] Installez les packages et donnez le rôle des packages suivants :
- php-gd : Contient des dépendances de la version PHP  
- php-mysqlnd : Contient des modules pour se connecter à MySQL avec des scritps php aisni que le module "mysql" qui permet de se connecter à toutes les versions MySQL  
- php-pdo : Contient un objet partagé dynamique qui ajoutera une couche d'abstraction d'accès à la base de données vers PHP, il fournit une interface commune pour accéder à MySQL, PostgreSQL ou autre bases de données.  
- php-mbstring : Contient un objet partagé dynamique qui ajoutera prise en charge de la gestion des chaînes multi-octets vers PHP.

## 8- Configuration avancée
### [8A] Installez le service vsftp, et bien sûr, vérifiez qu’il est opérationnel. Qu’avez vous fait comme commande ?
```bash
yum install -y vsftpd #installation
systemctl status vsftpd #vérification
```
### [8B] Créez un compte utilisateur nommé ‘web’.
```bash
useradd web
passwd web
cd home/web
mkdir www.campagne.sys www.montagne.sys www.glacier.sys www.sommet.sys
```
### [8C] Vérifiez que vous pouvez accéder en lecture/écriture a tous les répertoires présents dans le répertoire /home/web en utilisant un client ftp comme FileZilla. Donnez les permissions de /home/web ainsi que celles des dossiers précédemment créés

## 9-Configuration réseau :
### [9A] Configurer votre carte réseau avec l’IP fixe 172.16.xxx.5/24 et donnez le contenu du fichier
j'ai utilisé la commande **ntmui**
### [9B] A quoi sert la commande nmcli
nmcli sert à contrôler NetworkManager et signaler l'état du réseau
### [9C] A quoi sert le service NetworkManager
Network-Manager est l'outil de gestion des connexions réseau, il permet la création et la configuration des accès à divers types de réseaux
### [9D] A quoi sert le fichier /etc/resolv.conf ?
Ce fichier permet d'indiquer au système quels serveurs interroger pour résoudre un nom de domaine
### [9E] Que devez-vous faire comme tests pour vérifier la connectivité de votre VM ?
Il suffit de ping un site, par exemple **ping 8.8.8.8**
### [9F] Votre machine peut-elle joindre www.free.fr ? Quelle commande avez vous utilisez pour le vérifier ?
Oui cela fonctionne avec **ping www.free.fr**
### [9G] Votre VM peut-elle être jointe par l’un de vos collègues ? Si non, pourquoi ? Quels tests avez vous effectués ?
Non car vm est en local donc impossible que mes collègues me ping

## 10-IP Virtuelles
### [10A] Ajouter trois adresses IP virtuelles :
- 172.16.xxx.6/16 permettant de contacter le site web www.campagne.sysTYPE="Ethernet"  

BOOTPROTO="static"  
DEFROUTE="yes"  
NAME="ens33:0"  
DEVICE="ens33:0"  
ONBOOT="yes"  
IPADDR=192.168.12.137  
NETMASK=255.255.255.0  

- 172.16.xxx.10/16 permettant de joindre www.montagne.sys et www.glacier.sys  

TYPE="Ethernet"  
BOOTPROTO="static"  
DEFROUTE="yes"  
NAME="ens33:1"  
DEVICE="ens33:1"  
ONBOOT="yes"  
IPADDR=192.168.12.138  
NETMASK=255.255.255.0  

- 172.16.xxx.15/16 permettant de joindre www.sommet.sys  

TYPE="Ethernet"  
BOOTPROTO="static"  
DEFROUTE="yes"  
NAME="ens33:2"  
DEVICE="ens33:2" 
ONBOOT="yes"  
IPADDR=192.168.12.139  
NETMASK=255.255.255.0
### [10B] Comment prouver que votre configuration est fonctionnelle ?
En faisant des pings vers ces adresses IP

## 11- Configuration du premier serveur virtuel : serveur virtuel par IP
### [11A] Créez le premier serveur web virtuel. La racine (DocumentRoot) de ce serveur web sera bien sûr le répertoire www.campagne.sys. Donnez le nom et le contenu du fichier de configuration.
```bash
nano /etc/httpd/conf/httpd.conf
```
Listen 80  
<VirtualHost 192.168.12.137>  
DocumentRoot "/www/home/web/campagne.sys"  
ServerName www.campagne.sys  
</VirtualHost># Dynamic Shared Object (DSO) Support  
### [11B] Prouvez que votre serveur virtuel est a l’écoute et joignable depuis une machine présente sur le réseau vmnet8. Quelle est votre démarche ? 
On peut faire un ping
###  [11C] Testez votre configuration en utilisant votre navigateur (depuis votre Windows ou depuis une autre VM). Pouvez-vous prouver que c’est bien le serveur virtuel que vous venez de configurer qui a répondu ? Comment ?
Dans le fichier host de notre machine physique et on ajoute l'adresse ip et l'adresse web, puis on teste dans notre navigateur http://www.campagne.sys
### [11D] Votre serveur vous a retourné un message « access denied » ? C’est probablement normal, pourtant vous avez prouvé qu’il était a l’écoute :
Allow indique les services et les hôtes pour lesquels l'accès est autorisé
```bash
chmod 755 /home/web
```
### [11E] Déployez l’application web phpsysinfo (http://phpsysinfo.github.io/phpsysinfo/) dans le répertoire www.campagne.sys et fournissez une copie d’écran de la page d’accueil de cette application.
###  [11E] Testez l’URL http:// 172 .16. xxx .5 Avez vous obtenu une réponse ? Si Oui pourquoi ?
On a un résultat car www.campagne.sys est associé à l'ip

## 12- Serveur virtuel par nom
### [12A] Créer les deux serveurs webs virtuels répondant pour le domaine www.montagne.sys et www.glacier.sys sur l’IP 172.16.xxx.10 et donnez le contenu du/des fichier·s de configuration.
```bash
192.168.12.138 www.glacier.sys  
192.168.12.138 www.montagne.sys  

<VirtualHost 192.168.12.138:80>  
        DocumentRoot "/home/web/www.glacier.sys/"
        ServerName www.glacier.sys
</VirtualHost>

<VirtualHost 192.168.12.138:80>
        DocumentRoot "/home/web/www.montagne.sys/"
        ServerName www.montagne.sys
</VirtualHost>
```
### [12C] Mettez en place une solution permettant de garantir que le serveur virtuel hébergeant www.montagne.sys est bien le bon a répondre et procédez aux tests. Justifiez votre démarche.
###  [12D] Testez l’URL http://192.168.12.138 . Quel serveur virtuel a répondu ? Est-ce le comportement attendu ? Justifiez votre réponse. 
C'est glacier qui répond mais on voudrait avoir le choix entreles 2

## 13-Serveur virtuel, HTTPS et SSL
### [13A] Avez vous vérifié que le module nécessaire a l’utilisation du HTTPS était installé et chargé ? Quel est le nom du module ? qu’avez vous fait pour vérifier qu’il était chargé en mémoire par Apache ?
```bash
yum install ssl_mod
apachectl start
httpd -M
```
###  [13B] La commande « openssl req -x509 -nodes -newkey rsa:1024 -keyout serveur.cle -out certificat.pem » permet de générer les fichiers nécessaires pour l’utilisation de l’https.
-A quoi sert l’option « -x509 »  
C'est une norme spécifiant les formats

-A quoi sert le paramètre « rsa:1024 » de l’option « -newkey » ?  
Cest pour créer une clé publique chiffrée sur 1024 bits 

-Quels sont les fichiers générés ? Que contiennent-ils ? 
**serveur.cle** : la clé chiffrée
**certificat.pem** : un certificat chiffré
###  [13C] Mettez en place le serveur virtuel www.sommet.sys Il devra répondre sur les ports 80 et 443, et donnez le contenu du fichier de configuration.
```bash
<VirtualHost 192.168.12.139:443>
        DocumentRoot "/home/web/www.montagne.sys/"
        ServerName www.montagne.sys
</VirtualHost>
```
### [13D] Quel message avez-vous obtenu lors de vos tests avec votre navigateur ? Pourquoi avez vous obtenu ce message ?
### [13E] Connaissez vous le site Let’s Encrypt (https://letsencrypt.org/fr/). Quel est son rôle ? Comment peut-il vous aidez pour le problème précédent ?

## 14-Restriction de l’accès utilisateur
### [14A] Quels sont les modes d’authentification possibles?
### [14B] Qu’est ce qu’un fournisseur d’accès dans ce contexte?
### [14C] Pour le site www.campagne.sys, créer un répertoire « secure » et vérifier que vous pouvez bien accéder au contenu de ce répertoire. Quels tests avez-vous effectués ?
### [14D] Changer la configuration de ce serveur virtuel pour restreindre l’accès au répertoire « secure ». Vous utiliserez une authentification « basic » avec un « provider file». Donnez le contenu du fichier de configuration.[13D] Quel message avez-vous obtenu lors de vos tests avec votre navigateur ? Pourquoi avez vous obtenu ce message ?
### [14E] Créez un utilisateur « Apache » avec les identifiants « leon/123+aze » et vérifiez qu’il a bien accès au répertoire « secure » . La commande htpasswd vous sera nécessaire. Quels tests avez-vous effectués ? Quelles commandes avez-vous utilisées ?
### [14F] Quelle démarche vous a permis de vérifier que le compte « leon » avec le mot de passe « 123+aze » a bien accès au répertoire « secure » ?

## 15-Fichier .htaccess
### [15A] Qu’est ce qu’une réécriture d’URL ? Quel module est nécessaire pour faire une réécriture d’URL ?
### [15B] Qu’est ce qu’un Framework ?
### [15C] A quoi sert la directive AllowOverride ?
### [15D] Quels sont les 6 paramètres pris en compte par la directive AllowOverride ?
### [15E] Lequel des paramètres permet d’autoriser l’usage d’une réécriture d’URL a partir d’un fichier .htaccess ?
### [15F] Créez un répertoire « private » avec un fichier .htaccess. Donnez le contenu du fichier .htaccess
### [15G] Dans le fichier .htaccess précédent, mettez en place les directives nécessaires pour autoriser uniquement l’utilisateur « Apache gaston/123+aze » a accéder au répertoire. Quelle est votre démarche ? Quelles commandes avez vous utilisées ?
