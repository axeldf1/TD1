# TD4 – Programmation C Système / Les processus
## 3- Commandes de base
### [3A] Quel est le rôle des commandes suivantes
- ps : affiche les informations des processus actif
- pstree : affiche les processus sous forme d'arbre
- kill : termine un processus
### [3B] Tapez la commande ps -aux. Quelle est son utilité ? A quoi correspondent les colonnes USER, PID, %CPU,%MEM, VSZ, RSS, TTY, STAT, START, TIME
Cette commande permet de visualiser les utilisateurs associés à chaque processus ainsi que des informations suplémentaires tel que :
- USER : l'utilisateur qui a lancé le processus
- PID : numéro du processus
- %CPU : affiche l'utilisation du processeur en pourcentage
- %MEM : affiche l'utilisation de la mémoire vive en pourcentage
- VSZ : Virtual Memorie Sizedonne l'utilisation des bibliotheques partagées et la memoire utilisé pour son fonctionnement
- RSS : Resident Set Size donne l'utilisation de la mémoire physique utilisée
- TTY : indique le numéro de port du terminal (le "?" indique que la commande n'est pas associée à un terminal)
- STAT : affiche l'état actuel du processus
- START : indique l'heure a laquelle le processus a commencé
- TIME : affiche le temps processeur utilisé par ce processus
### [3C] La commande top ou htop affiche une colonne PR et NI. A quoi correspond les deux colonnes ? Quelle est la difference entre PR et NI ?
- PR : priorité 
- NI :indique La valeur de politesse du processus
Une valeur négative signifie une priorité plus haute, à l'inverse d'une valeur positive. Un zéro signifie simplement que le lancement d'une tâche ne tiendra pas compte de la priorité.
### [3D] Quelle commande permet d’afficher la priorité d’un processus
La commande **nice** permet d'afficher la priorité d'un processus
## [3E] Quelle commande permet de changer la priorité d’un processus
La commande **renice** permet de modifier la priorité d'un processus 
```bash
renice <Priorité> {-p <PID> | -g <GID> | -u <User>}
```
### [3F] Quelle est la différence entre kill -3 et kill -9 ? A quoi correspondent les options -3 et -9 ? Donnez la liste des principaux signaux ( valeur numérique, nom, rôle)
- kill -3 : SIGQUIT, ?
- kill -9 : SIGKILL, termine le processus  
![Tableau SIG](SIG.PNG)
### [3G] Quelle est la particularité du signal SIGKILL ?
Termine à coup sûr le processus
### [3H] Quel est le rôle de la commande nohup ?
Permet de lancer un processus qui restera actif même après la déconnexion de l'utilisateur
### [3I] Quelles commandes vous permettent de passer un processus en arrière plan ? De le ramener en avant plan ? De le mettre en pause ?
- **command &** permet de lancer le processus en arrière plan et **bg <PID>** permet de le passer en arrière plan
- **fg <PID>** permet de le ramener en avant plan
- **kill -STOP PID** permet de mettre en pause le processus

## 4- Gestion des processus
### [4A] Il existe deux approches pour passer le processus en « background » (tache de fond). Lesquelles ?
**./randomgenerator &** permet de lancer le processus en arrière plan et **bg <PID>** permet de le passer en arrière plan
### [4B] Votre processus est en tache de fond ? Tapez la commande « clear » ? Qu’observez vous ?
Le procesus est en fond et après **clear** le processus tourne encore
### [4C] Comment pouvez vous mettre le processus en pause ? Il existe deux approches, lequelles ?
On peut mettre en pause en faisant **CTRL + Z** ou **kill -STOP PID**
### [4D] Que devez vous faire pour le ramenez en « foreground » (avant plan?)
La commande **fg <PID>** permet de le ramener en avant plan
### [4E] Que devez vous faire pour arrêter le processus. Vous avez deux solutions, lesquelles ?
On peut arrêter le processus avec **KILL %1** ou **CTRL + C**
### [4F] Quelle est la différence entre un numéro de tache et un numéro de processus.
Le numéro de processus est global tant dis que le numéro de tâche est utilisé par le terminal uniquement
### [4G] Lancer le programme en tache de fond depuis un terminal ssh et déconnectez-vous du terminal. Le processus et il actif ? Comment avez vous vérifié?
Le processus est en sleep, on peut vérifier avec la commande **htop -p <PID>**
### [4H] Comment pouvez vous lancez un processus qui restera actif même si vous fermez la session ? Il existe au moins deux solutions, lesquelles ?
 Pour lancer un processus qui reste actif après la fermeture du terminal on peut faire :
 - **nohup command**
 - **screen -S LE_NOM_DU_SCREEN** puis on tape notre commande et on ferme le terminal
### [4I] Comment lancer randomgenerator avec une priorité plus faible ?
**nice -n 0 ./randomgenerator**
### [4J] Quelle commande vous permet de changer la priorité de randomgenerator ?
 **renice +10 2396**
 
## 5- Redirection de flux
### [5A] Que fait la commande randomgenerator > rands.txt ?
Cette commande est sensé écrire les résultats dans rands.txt au lieu de l'écrire dans la console, le fichier est écrasé si il existait déjà, cependant rien ne se passe dans notre cas
### [5B] Que fait la commande randomgenerator >> rands.txt ?
Cette commande fait la même chose que la précédente à l'exception qu'elle n'écrase pas le fichier et écrit à la fin
### [5C] Que fait la commande randomgenerator -i ?
Permet à l'utilisateur de renseigner le nombre de chiffre à générer
### [5D] Que fait la commande randomgenerator -i <<< "10"
Permet de générer 10 chiffres
### [5E] Tapez la commande echo "20" > response.txt Que fait-elle ?
Crée un fichier response.txt et écris 20 dedans, si le fichier existe il est écrasé
### [5F] Que fait la commande randomgenerator -i < response.txt
Permet de générer le nombre de chiffre inscrit dans "response.txt"
### [5G] Que fait la commande randomgenerator -i <<< "10" | sort -n
Génère 10 chiffres et les tri par odres croissant
### [5H] A quoi correspond les flux 0, 1, 2 (stdin, stdout, stderr) ?
- stdin : Entrée standard
- stdout : Objet de flux de sortie standard
- stderr : Erreur standard
### [5I] Que fait la commande randomgenerator -i <<< "50" > data.txt &
Permet de générer 50 chiffres et de les inscrire dans "data.txt"
### [5J] Vous devez maintenant avoir un fichier nommé data.txt, que fait la commande cat data.txt | sort -n
Permet de trier les chiffres de "data.txt" dans l'ordre croissant
### [5K] Modifier le code source de RandomGenerator pour qu'il accepte l'option -n XXX, XXX étant le nombre de valeurs aléatoires désirées, et donnez le code.
```C
int main(int argc, char** argv) {
    char oc;
    int repet = -1;

    while ((oc = getopt(argc, argv, "ian:")) != -1) {
        switch (oc) {
            case 'a':
                printf("boummm \n");
                break;
            case 'i':
                printf("nombre de valeur : ");
                scanf("%i", &repet);
                break;
            case 'n':
                repet = atoi(optarg);
            	break;
            case '?':
            default:
                break;
        }
    }

    while (repet-- != 0 ) {
        printf("%d\n", rand()%1000);
        usleep(500000);
    }
    return (EXIT_SUCCESS);
}
```
## 6- Création d'un processus en C
La commande fork() permet de "forker" (cloner) le processus courant.
### [6A] Quel intérêt a un processus de se 'forké' ? Dans quel TD précédent un processus est-il « forké » ? Pour qu'elles raisons ?
L'intérêt est de pouvoir faire une commande en parallèle, dans le TD 2 MariaDB est un fork de MySQL pour sécuriser l'accès futur à MySQL et son développement ultérieur
### [6B] Quelles sont les particularités d'un processus « forké »
- le processsus fils hérite de l’environnement de son père (environement d’exécution, variables-système)
- en fait il reçoit une copie
- à l’exception de son identité pid et de celle de son père ppid
### [6C] Quelle différence existe il entre un processus « forké » et un thread ?
Un thread partage la même mémoire virtuelle, tandis que les processus forké s’exécutent dans des espaces mémoire distincts
### [6D] Quelle est l'une des limites de l'utilisation de fork() ?
Un processsus, un ulilisateur, mais aussi le système ont un nombre limité de processus...
Si un programme alloue des processus en boucle, il risque de consommer toutes les resources (en processus) et de ne même plus pouvoir allouer de processus pour tuer le programme fautif
### [6E] Quels problèmes peuvent poser les threads ?
- De nombreux threads bloqués pendant une longue période
- Conflit de ressources partagées
- Threads dont l'exécution s'éternise

## 7- Utilisation de fork, wait, waitpid, sleep
### [7A] Que donne les commandes ps et pstree pendant l’exécution du programme ?
```bash
 PID TTY          TIME CMD
   2876 pts/1    00:00:00 bash
   2916 pts/1    00:00:00 test
   2917 pts/1    00:00:00 test
   2918 pts/1    00:00:00 test
   2920 pts/1    00:00:00 test
   2921 pts/1    00:00:00 test
   2922 pts/1    00:00:00 test
   2923 pts/1    00:00:00 test
   2929 pts/1    00:00:00 ps
```
### [7B] Observez l’état des processus.Que se passe-t-il ? Corriger le code du processus père pour que les processus se terminent correctement.
```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <time.h>
#include <sys/types.h>

static int global_int = 42;

void ChildProcess(int child_id)
{
    int time_to_sleep = rand() % 110 + 20;
    sleep(time_to_sleep);
    execlp("/home/student/Desktop/RANDOM/ICS_RandomGenerator/random.c","randomG",NULL );
    
    printf("Je suis le fils numéro %d PID %d et parent PID %d\n", child_id, getpid(), getppid());
    printf("variable globale : %d \n",global_int);
    global_int = rand();
    printf("variable globale : %d \n",global_int);
    
    exit(0);
}

void exit_handler(int ev, void *arg)
{
    fprintf(stdout, "bye \n");
}

int main(int argc, char **argv)
{
    atexit(exit_handler);
    int repet;
    struct timespec tp;
    clockid_t clk_id;
    printf("nombre de valeur : ");
    scanf("%i", &repet);
    
    if (repet < 10) {
        for (int i = 0; i < repet; i++)
        {
            clock_gettime(clk_id, &tp); 
            srand(tp.tv_nsec);
            if (fork() == 0) 
            {
                ChildProcess(i);
            }
        }
    }
    
    return (EXIT_SUCCESS);
}
```

