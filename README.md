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
