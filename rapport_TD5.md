*PRATOUSSY Martin - 3ICS G2*

# TD5 - Programmation C Système

---

**[1A]** -	
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
#include <sys/types.h>
#include <signal.h>
#include <string.h>
#include <time.h>

#define NB_CHILD 5

void sigaction_handler(){
    printf("Hello from %d\n", getpid());
}

void child_process(){  
    struct sigaction new_action;
    memset(&new_action, '\0', sizeof(new_action));
    new_action.sa_handler = sigaction_handler;
    new_action.sa_flags = 0;
    sigaction(SIGUSR1, &new_action, NULL);
    while(1);
    exit(EXIT_SUCCESS);
}

int main(int argc, char const *argv[]) {

    pid_t pid;
    pid_t id[NB_CHILD];
    for (int i = 0; i < NB_CHILD; i++) {
        pid = fork();
        id[i] = pid;
        if(pid == 0) {
            child_process();
        }
    }

    for (int i = 0; i < 30; i++) {
        srand(time(NULL));
        int id_to_send = rand()%NB_CHILD;
        kill(id[id_to_send], SIGUSR1);
        sleep(1);
    }

    for (int i = 0; i < 5; i++) kill(id[i], SIGINT);
    
    return (EXIT_SUCCESS);
}
```

**[1B]** -
La fonction **signal** est fortement déconseillée car elle ne bloque pas l'arrivée d'autres signaux pendant l'exécution du gestionnaire. Elle réinitialise aussi l'action du signal au signal par défaut (SIG_DFL), ce qui ouvre une fenêtre de vulnérabilité au moment de la réinstallation du gestionnaire. Le comportement du signal est aussi différent selon les systèmes.

---

**[2A]** -
```c
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

void sigaction_handler() {    
    printf("Enfant %d, le mémoire partagée contient la valeur %d\n", getpid(), *shared_memory);
}

void child_process(){  
    struct sigaction new_action;
    memset(&new_action, '\0', sizeof(new_action));
    new_action.sa_handler = sigaction_handler;
    new_action.sa_flags = 0;
    if(sigaction(SIGUSR1, &new_action, NULL) < 0) {
        printf("tenga tenga");
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
    pid = fork();
    pid_t id = pid;

    if(pid == 0) {
        child_process();
    }

    else if (pid > 0) {
        for(int i = 0; i < 5; i++) {
            int num = rand()%100;
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

---

**[3A]** -
La fonction **mmap** prend 6 paramètres :
- ```void *addr``` : adresse de départ de la nouvelle projection
- ```size_t length``` : longueur de la projection
- ```int prot``` : protection mémoire que l'on désire pour cette projection
- ```int flags``` : détermine si les modifications de la projection sont visibles par les autres processus projetant la même région et si les modifications sont appliquées au fichier sous-jacent
- ```int fd``` : descripteur de fichier
- ```off_t offset``` : position offset dans le fichier (doit être un multiple de la taille de page)

**[3B]** -
```PROT_READ | PROT_WRITE``` sont les valeurs passées à l'argument *prot* dans la fonction **mmap**, ils signifient qu'on peut lire et écrire dans la zone mémoire.

**[3C]** -
```MAP_SHARED``` est la valeur passée à l'argument *flag* dans la fonction **mmap**, il signifie que les modifications de la projection sont visibles par tous les autres processus qui projettent ce fichier.

**[3D]** -
**writer.c** :
```c
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
```c
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

**[3E]** -
TODO

---

**[4A]** -
```c
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

**[4B]** -
La fonction **dup2** prend en paramètres *oldfd* et *newfd*, elle permet de créer une copie du descripteur de fichier *oldfd* dans le nouveau descripteur de fichier spécifié par *newfd*. Les deux descripteurs font référence à la même description de fichier ouvert. 

**[4C]** -
- **popen** : engendre un processus en créant un pipe, en exécutant un fork et e, invoquant le shell. Il prend 2 arguments, *const char *commande* avec la commande à exécuter et *const char *type* qui spécifie si l'ouverture se fait en écriture ou en lecture.
- **pclose** : renvoie l"état de sortie de la commande exécutée lorsque le processus correspondant se termine.

---

**[5A]** -
Erreur lors de l'écriture dans le tube.
```c
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

**[5B]** -
Rien n'est lu dans le tube car **send_rand** ne parvient pas à y écrire.
```c
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

---

**[6A]** -
TODO

**[6B]** -
TODO
