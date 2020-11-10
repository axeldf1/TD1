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
### [7B] Le service Apache utilise désormais systématiquement le package ‘php-fpm’
contenant les éléments nécessaires pour l’exécution de pages web dynamiques. Que contient ce
package ?
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
``bash
php -v
```
on utilise la version : PHP 7.2.24
### [7E] Comment connaître la configuration compléte du serveur web (librairies installées,
avec leurs versions et configurations?)
```bash
php -info
```
### [7F] Installez les packages et donnez le rôle des packages suivants :
- php-gd : Contient des dépendances de la version PHP
- php-mysqlnd : Contient des modules pour se connecter à MySQL avec des scritps php aisni que le module "mysql" qui permet de se connecter à toutes les versions MySQL
- php-pdo : Contient un objet partagé dynamique qui ajoutera une couche d'abstraction d'accès à la base de données vers PHP, il fournit une interface commune pour accéder à MySQL, PostgreSQL ou autre bases de données.
- php-mbstring : Contient un objet partagé dynamique qui ajoutera prise en charge de la gestion des chaînes multi-octets vers PHP.

## 8- Configuration avancée
### [8A] Installez le service vsftp, et bien sûr, vérifiez qu’il est opérationnel. Qu’avez vous
fait comme commande ?
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
### [9A] Configurer votre carte réseau avec l’IP fixe 172.16.xxx.5/24 et donnez le contenu du
fichier.
### [9B] A quoi sert la commande nmcli
### [9C] A quoi sert le service NetworkManager
### [9D] A quoi sert le fichier /etc/resolv.conf ?
### [9E] Que devez-vous faire comme tests pour vérifier la connectivité de votre VM ?
### [9F] Votre machine peut-elle joindre www.free.fr ? Quelle commande avez vous utilisez
pour le vérifier ?
### [9G] Votre VM peut-elle être jointe par l’un de vos collègues ? Si non, pourquoi ? Quels
tests avez vous effectués ?
