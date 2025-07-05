
# Apuntes de Comunicación entre Procesos e Hilos en C (Linux clásico)

---

## ¿Cuándo usar qué?

| Herramienta          | Tipo de ejecución       | Comunicación       | Memoria compartida | Uso típico                                        |
|----------------------|-------------------------|--------------------|---------------------|--------------------------------------------------|
| `fork()`             | Proceso nuevo           | ❌                | ❌                 | Crear procesos aislados                          |
| `pipe()`             | Padre-hijo              | ✅ Unidireccional | ❌                 | Comunicación básica entre procesos relacionados  |
| `FIFO (mkfifo)`      | Procesos independientes | ✅ Unidireccional | ❌                 | Comunicación persistente vía archivo             |
| `pthread`            | Hilos (mismo proceso)   | ✅ por variables  | ✅                 | Concurrencia dentro del mismo proceso            |
| `signal()`           | Procesos                | ❌ (evento)       | N/A                | Notificaciones asincrónicas                      |
| `msgqueue` (SysV)    | Procesos independientes | ✅ estructurada   | ❌                 | Comunicación asincrónica estructurada            |

---

## Cómo crear y usar cada uno

---

### `fork()` – Crear procesos

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    pid_t pid = fork();

    if (pid == 0)
        printf("Soy el hijo\n");
    else
        printf("Soy el padre\n");

    return 0;
}
```

---

### `pipe()` – Comunicación entre padre e hijo

```c
#include <unistd.h>
#include <stdio.h>

int main() {
    int fd[2];
    pipe(fd);

    if (fork() == 0) { // hijo
        close(fd[1]);
        char buffer[100];
        read(fd[0], buffer, sizeof(buffer));
        printf("Hijo recibe: %s\n", buffer);
    } else { // padre
        close(fd[0]);
        write(fd[1], "Hola hijo", 10);
    }

    return 0;
}
```

---

### `FIFO (Named Pipe)` – Comunicación entre procesos separados

**Crear FIFO desde terminal:**

```bash
mkfifo myfifo
```

**Escribir (proceso A):**

```c
int fd = open("myfifo", O_WRONLY);
write(fd, "mensaje", 8);
close(fd);
```

**Leer (proceso B):**

```c
int fd = open("myfifo", O_RDONLY);
char buffer[100];
read(fd, buffer, sizeof(buffer));
printf("Leído: %s\n", buffer);
```

---

### `pthread` – Crear hilos

```c
#include <pthread.h>
#include <stdio.h>

void* funcion(void* arg) {
    printf("Hola desde el hilo\n");
    return NULL;
}

int main() {
    pthread_t hilo;
    pthread_create(&hilo, NULL, funcion, NULL);
    pthread_join(hilo, NULL);
    return 0;
}
```

---

### `signal()` – Notificaciones entre procesos

```c
#include <signal.h>
#include <stdio.h>
#include <unistd.h>

void handler(int sig) {
    printf("Señal recibida: %d\n", sig);
}

int main() {
    signal(SIGUSR1, handler);
    pause();  // espera señal
    return 0;
}
```

---

### `Colas de mensajes (System V)`

**Estructura del mensaje:**

```c
struct msgbuf {
    long mtype;
    char mtext[100];
};
```

**Crear clave y cola:**

```c
key_t key = ftok("archivo.txt", 'A');
int msqid = msgget(key, IPC_CREAT | 0666);
```

**Enviar mensaje:**

```c
struct msgbuf msg = {1, "Hola mundo"};
msgsnd(msqid, &msg, strlen(msg.mtext) + 1, 0);
```

**Recibir mensaje:**

```c
struct msgbuf msg;
msgrcv(msqid, &msg, sizeof(msg.mtext), 1, 0);
printf("Mensaje recibido: %s\n", msg.mtext);
```

**Eliminar la cola:**

```c
msgctl(msqid, IPC_RMID, NULL);
```

---

## Resumen comparativo

| Herramienta | Relación necesaria       | Comunicación estructurada | Memoria compartida | Ideal para...                               |
|-------------|--------------------------|----------------------------|--------------------|---------------------------------------------|
| `fork()`    | No                       | No                         | No                 | Separar tareas o procesos externos          |
| `pipe()`    | Padre-hijo               | No                         | No                 | Comunicación simple y lineal                |
| `FIFO`      | No                       | No                         | No                 | Comunicación entre procesos separados       |
| `pthread`   | Mismo proceso            | Por variables              | Sí                 | Paralelismo eficiente con bajo overhead     |
| `signal()`  | No                       | No                         | N/A                | Alertas o control de flujo entre procesos   |
| `msgqueue`  | No                       | Sí                         | No                 | Comunicación asincrónica y estructurada     |


---

### Cómo enviar señales

Puedes enviar señales entre procesos usando la función `kill()` desde C o el comando `kill` desde terminal.

#### Enviar señal desde C:

```c
#include <signal.h>
#include <unistd.h>

int main() {
    pid_t pid = /* PID del proceso destino */;
    kill(pid, SIGUSR1);  // Envia SIGUSR1 al proceso destino
    return 0;
}
```

#### Enviar señal desde terminal:

```bash
kill -SIGUSR1 <PID>
```

Puedes encontrar el PID con comandos como:

```bash
ps aux | grep nombre_programa
```

#### Señales comunes:

| Señal     | Código      | Descripción                         |
|-----------|-------------|-------------------------------------|
| `SIGINT`  | `2`         | Interrupción (Ctrl + C)             |
| `SIGTERM` | `15`        | Terminación programada              |
| `SIGKILL` | `9`         | Finalización forzada (no bloqueable)|
| `SIGUSR1` | `10` / `30` | Definida por el usuario             |
| `SIGHUP`  | `1`         | Reinicio del proceso o conexión     |

---
