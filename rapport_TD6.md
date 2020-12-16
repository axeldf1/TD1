# TD6 – Programmation C Système 3 : Gestion des fichiers
## 1 Les périphériques
### [1A] A quoi correspondent ces périphériques ?
- /dev/null : un périphérique qui supprime toutes les données qui y sont écrites
- /dev/zero : un pseudo périphérique spécial qui renvoie uniquement le caractère null lors d'une lecture
- /dev/random : un fichier spécial qui sert de générateur de nombres aléatoires particulièrement adapté pour les cas où l'on a ponctuellement besoin de nombres hautement aléatoires
- /dev/urandom : un fichier spécial qui sert de générateur de nombres aléatoires mais vulnérables à unecryptanalyse
- /dev/loop0 : un fichier spécial de périphérique de type bloc virtuel à utiliser
### [1B] En utilisant les fonctions open (man 3 open ) et read (man 3 read), écrivez un programme qui affiche un nombre aléatoire sur un int
```C
#include <unistd.h>
#include <fcntl.h>

void main() {
	 

	for (int i = 0; i < 5; i++) {
			printf("Valeur aléatoire %d : %ld \n", i, values[i]);
	}
}
```

##2 Le système de fichiers /proc
### [2A] A quoi correspondent les répertoires nommés par des numéros ?
Ces répertoires correspondent aux processus et leur PID

### [2B] A quoi correspond le fichier cmdline a l’intérieur d'un de ces répertoires ?
Ce fichier contient la ligne de commande complète du processus, sauf si le processus est un zombie

### [2C] A quoi correspond le fichier cwd a l’intérieur d'un de ces répertoires ?
Il s'agit d'un lien symbolique sur le répertoire de travail courant du processus

### [2D] A quoi correspond le fichier exe a l’intérieur d'un de ces répertoires ?
Ce fichier est un lien symbolique représentant le chemin réel de la commande en cours d'exécution

### [2E] A quoi correspond le fichier root a l’intérieur d'un de ces répertoires ?
Ce fichier est un lien symbolique qui pointe sur le répertoire racine du processus

### [2F] Que contient le fichier /proc/devices ?
Ce fichier affiche les divers périphériques d'entrée-sortie de caractères et périphériques blocs actuellement configurés

### [2G] Que pouvez vous dire du répertoire /proc/self ?
C'est un lien symbolique vers le répertoire de /proc correspondant au processus courant. La destination du lien /proc/self dépend du processus qui l'utilise: chaque processus voit son propre répertoire comme destination du lien

## 3- Test des permissions d’accès a un fichier

## 4- Verrous et opérations sur les fichiers
### [4A] Pourquoi peut-on avoir besoin de verrouiller un fichier en écriture ?

### [4B] Écrire un programme qui écrit des valeurs aléatoires dans un fichier. Ce programme devra placer un verrou en écriture (F_WRLCK) sur le fichier. Que ce passe-t-il si vous exécutez deux fois le même programme depuis 2 shells différents ?
```C
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <stdio.h>
#include <string.h>

void main() {

	int fd = open("/dev/random", O_RDONLY), values;
	read(fd, &values, sizeof values);
	close(fd);

	if (values != NULL) {

		int fd2 = open("/home/student/Desktop/test.txt", O_RDWR);
		struct flock fl;

		fl.l_type = F_WRLCK;
		fl.l_whence = SEEK_SET;
		fl.l_start = 0;
		fl.l_len = 32;

		if (fd2 == -1) {
			printf("Fichier n'existe pas\n");
		}

		if (fcntl(fd2, F_SETLK, &fl) == -1) {

			if (errno == EACCES || errno == EAGAIN) {
				printf("Déjà en cours d'utilisation\n");
			} else {
				printf("Fichier non trouvé \n");
			}

		} else {
			char *result;

			asprintf(&result, "%ld", values);

			fl.l_type = F_UNLCK;
			fl.l_whence = SEEK_SET;
			fl.l_start = 0;
			fl.l_len = 32;

			if (fcntl(fd2, F_SETLK, &fl) == -1) {
				printf("Handle error 2\n");
			} else {
				write(fd2, result, strlen(result));
			}
		}
		close(fd2);
	}
}
```
## 5- Buffers (Mémoire tampon)

## 6 Parcours d'un répertoire
### [6A] En utilisant les fonctions précédentes, créez un programme qui affiche la taille cumulée de tous les fichiers contenus dans un répertoire.
```C
#include <stdlib.h>
#include <unistd.h>
#include <fcntl.h>
#include <errno.h>
#include <stdio.h>
#include <dirent.h>
#include <sys/stat.h>

void main()
{
    struct dirent *file;
    int entitiesSize;
    struct stat stats;
    char *directoryPath = "/home/student/Desktop/C machine/TD6/src";
    DIR *directory = opendir(directoryPath);
    if (directory != NULL)
        while (file = readdir(directory))
        {
            char *filePath;
            asprintf(&filePath, "%s/%s", directoryPath, file->d_name);
            if (stat(filePath, &stats) == 0)
            {
                entitiesSize += stats.st_size;
                printf("filePath : %s | fileSize : %d \n", filePath, stats.st_size);
            }
        }
    closedir(directory);
    printf("la taille du dossier %s | est de : %d \n", directoryPath, entitiesSize);
}
```

## 7 Descripteurs de fichiers et fichiers binaires
### [7A] Quelles sont les différentes valeurs (avec leurs symboles) possibles pour le paramètre flags ?
### [7B] Que permet de faire le paramètre mode ?
### [7C] En utilisant les fonctions précédentes, écrire un programme qui sauvegarde la valeur 19496893802562113L dans un fichier binaire. Ouvrez le fichier. Qu'observez-vous ?
On observe que cette valeur est plus grande qu'un byte, elle est donc sauvegardée sur 2 bytes
### [7D] De même, créez un programme qui enregistre la valeur 0x4142434451525354L dans un fichier, en utilisant les fonctions précédentes. Affichez la valeur avec un printf en décimal et hexadécimal ? Que contient le fichier binaire ?
### [7E] Enregistrez la valeur précédente dans un fichier en utilisant la fonction fprintf. Que constatez-vous ?
### [7F] Quelle est la différence essentielle entre un fichier binaire et un fichier texte ?
### [7G] Que pouvez-vous dire du principe 'little endian' et 'big endian' ?
### [7H] A quelle groupe appartiennent les processeurs de la famille des Intel/AMD ?
### [7I] Donnez un modèle de processeur appartenant a l'autre groupe.
### [7J] Il existe d'autres fonctions permettant de lire et d’écrire dans un fichier, qui sont respectivement fread et fwrite. Quelles sont les différences entre read et fread ou write et fwrite ?
### [7K] Quelles informations importantes pouvez vous tirer du code précédent ?
### [7L] En utilisant fwrite, écrire un programme qui enregistre 100 valeurs (de 0 a 100) de
type int en binaire dans un fichier et les affiche simultanément. Que pouvez vous observer dans le fichier ?
### [7M] Écrire un second programme qui lit les valeurs précédentes du fichier et les affiche ?

## 8- Fichiers séquentiels et fichiers a accès direct
### [8A] Écrivez un programme qui enregistre les valeurs de 10 a 30 dans un fichier binaire.
### [8B] Votre programme doit ensuite relire les données stockées a raison d’une valeur sur trois (vous devez utiliser lseek)
### [8C] Maintenant votre programme doit, en plus, lire la 5ieme valeur enregistrée dans le fichier.

## 9-Sauvegarde d'une structure
### [9A] Créez un tableau de 4 « Personne »
### [9B] Affichez les données stockées dans les structures (nom, age et poids de chaque « Personne »)
### [9C] Créez une fonction qui sauvegarde le contenu du tableau dans un fichier binaire.
### [9D] Créez une fonction qui lit les données du fichier précédent et les affiche au fur et a mesure de la lecture. Vous devez bien sur retrouver les données qui étaient stockées dans les structures.
### [9E] Créez une fonction qui lit les données du fichier précédents et les stocke dans un tableau de « Personne »
### [9F] Même exercice que précédemment, mais cette fois avec la structure suivante :
