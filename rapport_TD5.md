# TD5 – Programmation C Système 2 : Communication inter-processus
## 1 Signaux
```C
#include <signal.h>
#include <stdio.h>
#include <stdlib.h>
#include <sys/types.h>
#include <unistd.h>

void sigusr1(){
    signal(SIGHUP, sigusr1);
    printf("Hello from PID %d \n", getpid());
}

void sigint(){
    signal(SIGINT, sigint);
    printf("Child with PID : %d SIGINT \n", getpid());
}

void ProcessChild(int id){
    printf("ID : %d - child PID : %d - PPID : %d \n", id, getpid(), getppid());
    
    signal(SIGUSR1, sigusr1);
    signal(SIGINT, sigint);
    
    while(1);
    exit(EXIT_SUCCESS);
}

void main(){
    for (int i = 0; i < 5; i++){        
        if (fork() == 0){
            ProcessChild(i);
        }
        
        sleep(1);
        
        if (childProcess > 0 && i == 4){
        
            for (int j = 0; j < 30; j++){
                printf("msg nb : %d \n", j);
                signal(SIGUSR1,sigusr1);
                kill(childProcess - rand() % 5, SIGUSR1);
                sleep(1);
            }
            
            for (int h = 0; h < 5; h++){
                printf("SINGINT  %d \n", h);
                signal(SIGINT, sigint);
                kill(childProcess - h, SIGINT);
                sleep(1);
            }
        }
    }
}
```

### [1B] Il existe une fonction signal qui permet de faire presque la même chose que sigaction. Pourtant son utilisation est fortement déconseillé. Pourquoi ?
La fonction **signal** est similaire à **sigaction** mais elle est déconseillée car elle ne bloque pas l'arrivée d'autres signaux. De plus, elle réinitialise l'action du signal, ce qui rend vulnérable au moment de la réinstallation du gestionnaire

## 2- Mémoire partagée
### [2A] En vous basant le fichier TD5-shared_memory.c et en réutilisant le code du TD sur les processus (ou celui de l’exercice précédent) vous devez :
- Ecrire un programme qui se « forke » (un seul processus fils)
- Le processus parent devra afficher un nombre aléatoire compris entre 0 et 100 et l’écrire
dans la mémoire partagée.
- Après-quoi le processus parent devra envoyer un signal SIGUSR1 au processus fils pour lui
signaler que des données sont disponibles dans la mémoire partagée.
- Le processus fils devra lire les données disponibles et afficher son pid en même temps que
les données lues.
- Après l’envoi de 5 valeurs aléatoires au processus fils, le processus parent mettra fin au
processus fils, et se terminera.
```C
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <signal.h>
#include <string.h>
#include <time.h>
#include <sys/shm.h>
#include <sys/stat.h>

int* shared_memory;
int segment_id;

void sigaction_handler(){    
    printf("Enfant PID %d, mémoire partagée : %d\n", getpid(), *shared_memory);
}

void child_process(){  
    struct sigaction action;
    memset(&action, '\0', sizeof(action));
    action.sa_handler = sigaction_handler;
    action.sa_flags = 0;
    
    if(sigaction(SIGUSR1, &action, NULL) < 0){
        printf("sigaction");
        exit(EXIT_SUCCESS);
    }
    
    while(1);
    exit(EXIT_SUCCESS);
}

void fn_shared_memory(){
    segment_id = shmget(IPC_PRIVATE, sizeof(int), IPC_CREAT | IPC_EXCL | S_IRUSR | S_IWUSR);
    if (segment_id < 0) {
        printf("*** shmget error (server) ***\n");
        exit(EXIT_FAILURE);
    }

    shared_memory = (int*)shmat(segment_id, NULL, 0);
    if ((int*)shared_memory == -1) {
        printf("*** shmat error (server) ***\n");
        exit(EXIT_FAILURE);
    }
}

int main(int argc, char const *argv[]) {

    srand(time(NULL));

    fn_shared_memory();

    pid_t pid;
    pid_t id = pid;

    if(fork() == 0) {
        child_process();
    }

    else if (pid > 0) {
        for(int i = 0; i < 5; i++) {
            int num = rand() % 100;
            *shared_memory = num;
            kill(id, SIGUSR1);
        }
        kill(id, SIGINT);
    }

    else {
        printf("*** Fork error ***");
        exit(EXIT_FAILURE);
    }

    return (EXIT_SUCCESS);
}
```
## 3- Mémoire mappée
### [3A] Pour mettre en correspondance un fichier ordinaire avec la mémoire d’un processus, utilisez l’appel mmap. Cette fonction accepte 6 paramètres. Donnez le rôle de chacun des paramètres avec les valeurs possibles
La fonction **mmap** prend 6 paramètres :
- ```void *addr``` : adresse de départ de la nouvelle projection
- ```size_t length``` : longueur de la projection
- ```int prot``` : protection mémoire que l'on désire pour cette projection
- ```int flags``` : détermine si les modifications de la projection sont visibles par les autres processus projetant la même région et si les modifications sont appliquées au fichier sous-jacent
- ```int fd``` : descripteur de fichier
- ```off_t offset``` : position offset dans le fichier (doit être un multiple de la taille de page)

### [3B] Que signifie la ligne « PROT_READ | PROT_WRITE » dans le fichier reader.c
```PROT_READ | PROT_WRITE``` sont les valeurs passées à l'argument *prot* dans la fonction **mmap**, ils signifient qu'on peut lire et écrire dans la zone mémoire.

### [3C] A quoi sert le drapeau MAP_SHARED ?
```MAP_SHARED``` est la valeur passée à l'argument *flag* dans la fonction **mmap**, il signifie que les modifications de la projection sont visibles par tous les autres processus qui projettent ce fichier.

### [3D] Utilisez les fichiers TD5-reader.c et TD5-writer.c pour écrire deux programmes. Le premier programme écrira sous forme binaire le contenu d’un tableau d’entiers de 5 valeurs aléatoires dans la mémoire mappée. Le second programme devra lire ces valeurs depuis la mémoire mappée, et les afficher.
**writer.c** :
```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

#define NB_VALUES 5
#define FILE_LENGTH 0x100

int random_range(unsigned const low, unsigned const high) {
    unsigned const range = high - low + 1;
    return low + (int) (((double) range) * rand() / (RAND_MAX + 1.0));
}

int writer(char *file) {

    int fd;
    int* file_memory;
    srand(time(NULL));
    fd = open(file, O_RDWR | O_CREAT, S_IRUSR | S_IWUSR);
    lseek(fd, FILE_LENGTH + 1, SEEK_SET);
    write(fd, " ", 1);
    lseek(fd, 0, SEEK_SET);
    file_memory = mmap(0, FILE_LENGTH, PROT_WRITE, MAP_SHARED, fd, 0);
    close(fd);
    for (int i = 0; i < NB_VALUES; i++) {
        file_memory[i] = random_range(-100, 100);
    }
    
    return EXIT_SUCCESS;
}

int main(char argc, char const *argv[]) {

    if (writer("/home/student/Desktop/mapped_memory.bin") < 0) {
        printf("*** Writer error ***\n");
    }

    return(EXIT_SUCCESS);
}
```
**reader.c** :
```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <unistd.h>

#define NB_VALUES 5
#define FILE_LENGTH 0x100

int reader(char *file) {
    int fd;
    int* file_memory;
    int integer;

    fd = open(file, O_RDWR, S_IRUSR | S_IWUSR);
    file_memory = mmap(0, FILE_LENGTH, PROT_READ | PROT_WRITE, MAP_SHARED, fd, 0);
    close(fd);
    for (int i = 0; i < NB_VALUES; i++) {
        printf(" valeur : %d\n ", file_memory[i]);
    }
    return 0;
}

int main(char argc, char const *argv[]) {

    if (reader("/home/student/Desktop/mapped_memory.bin") < 0) {
        printf("*** Reader error ***\n");
    }

    return(EXIT_SUCCESS);
}
```

### [3E] Que se passe-t-il si le fichier de « mapping » se trouve sur un partage réseau NTFS ou SMB ? Quelles perspectives entrevoyez vous dans l’usage d’une mémoire mappé par rapport a une mémoire partagée ?

## 4- Tubes (aka « pipes »)
### [4A] Vous devez écrire un programme qui se « forke » :
- Le processus parent devra envoyer « n » valeurs aléatoires comprises entre 0 et 9 au
processus fils a travers un tube. La valeur de « n » est comprise entre 5 et 20.
- Le processus fils devra lire les toutes les valeurs qui lui sont transmises, calculer
leurs somme et quitter.
- Le processus parent doit se terminer lorsque le processus fils a fini sa tache.
```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

int pipe_fds[2];

void child_process() {
    int pipevalues;
    int sum = 0;
    close(pipe_fds[1]);
    while(read(pipe_fds[0], &pipevalues, sizeof(pipevalues)) != 0) {
        sum += pipevalues;
    }
    printf("Somme des valeurs de la pipe : %d\n", sum);
    close(pipe_fds[0]);
    exit(EXIT_SUCCESS);    
}

int main(int argc, char const *argv[]) {

    srand(time(NULL));

    if (pipe(pipe_fds) == -1) {
        printf("*** Pipe error ***");
        exit(EXIT_FAILURE);
    }

    pid_t pid = fork();

    if (pid == -1) {
        printf("*** Fork error ***");
        exit(EXIT_FAILURE);
    }
    else if (pid == 0) {
        sleep(3);
        child_process();
    }
    else {
        close(pipe_fds[0]);
        sleep(1);
        int n = rand() % 16 + 5;
        for (int i = 0; i < n; i++) {
            int randomnumber = rand() % 9;
            printf("Random number : %d\n", randomnumber);
            write(pipe_fds[1], &randomnumber, sizeof(randomnumber));
        }
        close(pipe_fds[1]);
    }
    
    return(EXIT_SUCCESS);
}
```
### [4B] Quelle est le rôle/intérêt de la commande dup2 ?
La fonction **dup2** prend en paramètres *oldfd* et *newfd*, elle permet de créer une copie du descripteur de fichier *oldfd* dans le nouveau descripteur de fichier spécifié par *newfd*. Les deux descripteurs font référence à la même description de fichier ouvert. 

### [4C] A quoi servent les commandes popen et pclose ?
- **popen** : engendre un processus en créant un pipe, en exécutant un fork et e, invoquant le shell. Il prend 2 arguments, *const char *commande* avec la commande à exécuter et *const char *type* qui spécifie si l'ouverture se fait en écriture ou en lecture.
- **pclose** : renvoie l"état de sortie de la commande exécutée lorsque le processus correspondant se termine.
## 5- Tubes nommés (FIFO, aka « named pipe »)
### [5A] Vous devez écrire un premier programme appelé send_rand qui écrit « n » valeurs aléatoires dans un tube nommé. Ce programme prendra en argument le nom du tube, et l'option -n qui sera suivi du nombre de valeurs aléatoires a envoyer dans le tube. Ce programme devra créer le tube dans le répertoire /tmp si celui-ci n'existe pas déjà.

Erreur lors de l'écriture dans le tube.
```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

extern char* optarg;
#define PIPE_PATH "/tmp/fifo"

int main(int argc, char const *argv[]) {

    srand(time(NULL));

    int fp = open(PIPE_PATH, O_RDWR, S_IRUSR | S_IWUSR);

    if (mkfifo(PIPE_PATH, S_IRWXU | 0666) == -1) {
        printf("*** FIFO error ***");
        exit(EXIT_FAILURE);
    }

    int c, n;
    while ((c = getopt(argc, argv, "n:")) != -1) {
        switch (c) { 
        case 'n':
            n = atoi(optarg);
            break;
        case '?':
            printf("*** Error: Unknown option ***\n");
            break;
        default:
            break;
        }
    }

    int* values = malloc(sizeof(int)*n);
    for (int i = 0; i < n; i++) {
        values[i] = rand() % 100;
        printf("Random number : %d\n", values[i]);
    }
    if (write(fp, values, sizeof(int)*n) == -1) {
        printf("*** Write error ***");
    }
    close(fp);

    return(EXIT_SUCCESS);
}
```
### [5B] Le second programme appelé get_rand lira toutes les valeurs présentes dans le tube, et affichera la moyenne des valeurs avant de quitter.
Rien n'est lu dans le tube car **send_rand** ne parvient pas à y écrire.
```C
#include <stdlib.h>
#include <stdio.h>
#include <fcntl.h>
#include <sys/mman.h>
#include <sys/stat.h>
#include <time.h>
#include <unistd.h>

#define PIPE_PATH "/tmp/fifo"

int main(int argc, char const *argv[]) {

    FILE* fp;
    fp = fopen(PIPE_PATH, "r");

    int value = 0, average, i = 1;
    while (read(fp, &value, sizeof(value)) != 0) {
        value += value;
        i++;
    }
    average /= i;

    printf("Moyenne des valeurs : %d\n", average);
    
    fclose(fp);

    return(EXIT_SUCCESS);
}
```
## 6- Socket
**[6A]** -
TODO

**[6B]** -
TODO
