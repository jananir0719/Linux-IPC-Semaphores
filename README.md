# Linux-IPC-Semaphores
Ex05-Linux IPC-Semaphores

# AIM:
To Write a C program that implements a producer-consumer system with two processes using Semaphores.

# DESIGN STEPS:

### Step 1:

Navigate to any Linux environment installed on the system or installed inside a virtual environment like virtual box/vmware or online linux JSLinux (https://bellard.org/jslinux/vm.html?url=alpine-x86.cfg&mem=192) or docker.

### Step 2:

Write the C Program using Linux Process API - Sempahores

### Step 3:

Execute the C Program for the desired output. 

# PROGRAM:

## Write a C program that implements a producer-consumer system with two processes using Semaphores.
```
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/ipc.h>
#include <sys/shm.h>
#include <sys/sem.h>
#include <sys/types.h>
#include <wait.h>

#define BUFFER_SIZE 5

// Semaphore union (required)
union semun {
    int val;
};

// Semaphore operations
void sem_wait(int semid, int semnum) {
    struct sembuf sb = {semnum, -1, 0};
    semop(semid, &sb, 1);
}

void sem_signal(int semid, int semnum) {
    struct sembuf sb = {semnum, 1, 0};
    semop(semid, &sb, 1);
}

int main() {
    int shmid, semid;
    int *buffer;
    int in = 0, out = 0;

    // Create shared memory
    shmid = shmget(IPC_PRIVATE, BUFFER_SIZE * sizeof(int), IPC_CREAT | 0666);
    buffer = (int *)shmat(shmid, NULL, 0);

    // Create 3 semaphores: mutex, empty, full
    semid = semget(IPC_PRIVATE, 3, IPC_CREAT | 0666);

    union semun s;

    // Initialize semaphores
    s.val = 1;              // mutex
    semctl(semid, 0, SETVAL, s);

    s.val = BUFFER_SIZE;    // empty
    semctl(semid, 1, SETVAL, s);

    s.val = 0;              // full
    semctl(semid, 2, SETVAL, s);

    if (fork() == 0) {
        // Producer
        for (int i = 1; i <= 10; i++) {
            sem_wait(semid, 1);   // wait(empty)
            sem_wait(semid, 0);   // wait(mutex)

            buffer[in] = i;
            printf("Produced: %d\n", i);
            in = (in + 1) % BUFFER_SIZE;

            sem_signal(semid, 0); // signal(mutex)
            sem_signal(semid, 2); // signal(full)

            sleep(1);
        }
        exit(0);
    } 
    else {
        // Consumer
        for (int i = 1; i <= 10; i++) {
            sem_wait(semid, 2);   // wait(full)
            sem_wait(semid, 0);   // wait(mutex)

            int item = buffer[out];
            printf("Consumed: %d\n", item);
            out = (out + 1) % BUFFER_SIZE;

            sem_signal(semid, 0); // signal(mutex)
            sem_signal(semid, 1); // signal(empty)

            sleep(2);
        }

        wait(NULL);

        // Cleanup
        shmctl(shmid, IPC_RMID, NULL);
        semctl(semid, 0, IPC_RMID);
    }

    return 0;
}
```



## OUTPUT
$ ./sem.o 
<img width="447" height="602" alt="Screenshot from 2026-02-19 09-01-23" src="https://github.com/user-attachments/assets/c392029e-8149-47e8-b54d-f920df70a9d8" />


$ ipcs





# RESULT:
The program is executed successfully.
