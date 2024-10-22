### ¿Qué política de planificación utiliza `xv6-riscv` para elegir el próximo proceso a ejecutarse?


La politica que utiliza es *Round Robbin* ejectutando una lista de proceso donde cada      se ejecuta en un tiempo fijo antes de ser interrumpido y ejecutar el sieguiente.



###   ¿Cúales son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar de estado?

```c
     enum procstate { UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };
```

-**UNUSED** : El proceso no esta en uso.
-**USED** :  El proceso esta en uso.
-**SLEEPING** : El proceso esta durmiendo, Esta esperando que ocurra un evento.
-**RUNNEABLE** :  El proceso está listo para ser ejecutado. (READY).
-**RUNNING** : El proceso está en ejecución.
-**ZOMBIE**:  El proceso ha terminado y está esperando a que su padre recoja su estado.


- Las funciones que hacen cambiar de estado a los procesos son las siguientes:

  - **procinit** : Inicializa el arreglo de procesos cambiando sus estados a UNUSED:

  ``` c
   (p->state = UNUSED)
  ```

  - **allocproc** : Cambia el estado de un proceso a USED:

  ```c
  (p->state = USED)
  ```

  - **freeproc** : Libera un proceso dejando su estado en UNUSED: 

  ```c
  (p->state = UNUSED)
  ```

  - **userinit** : Deja un proceso listo para correr, es decir, cambia el estado de un proceso a RUNNABLE:

  ```c
  (p->state = RUNNABLE) 
  ```

  - **fork** : Esta funcion puede allocar un proceso a traves de *allocproc*, dejarlo listo para correr a traves de *userinit* o liberarlo si algun paso no funciono a traves de *freeproc*.  

  - **exit** : Cambia su estado a ZOMBIE para que el proceso padre lo pueda recoger: 

  ```c
  (p->state = ZOMBIE)
  ```

  - **scheduler** : Al primer proceso que encuentre con el estado RUNNABLE en el arreglo de procesos lo cambia a RUNNING. 

  - **yield** : Cambia el estado de un proceso en ejecucion, es decir:

  ```c
  /* Modifica de */ (p->state = RUNNING) 
   /* a */          (p->state = RUNNABLE)
  ```

  - **sleep** : Manda a dormir a un proceso, es decir, cambia su estado a SLEEPING 

  ```c 
  (p->state SLEEPING)
  ```

  - **wakeup** : Despierta todos los procesos de un canal y cambia su estado de SLEEPING a RUNNABLE. 

  - **kill** : Necesita despertar a un proceso en caso de estar dormido para verificar si se debe eliminar, es decir cambia el estado de un proceso de SLEEPING A RUNNABLE. 


### ¿Qué es un *quantum*? ¿Dónde se define en el código? ¿Cuánto dura un *quantum* en xv6-riscv?

#### ¿Qué es un quantum?

Un quantum es una unidad de tiempo durante la cual un proceso se ejecuta en un sistema de tiempo compartido. Al finalizar el quantum, si el proceso no ha terminado, el sistema operativo interrumpe su ejecución y da la oportunidad a otro proceso de ejecutarse. Esto permite la multitarea y asegura que todos los procesos reciban tiempo de CPU.

#### ¿Dónde se define en el código?

 El quantum se define en:
- *Archivo*: start.c
- *Función*: timerinit
- *Línea relevante*: 
```c
int interval = 1000000; // cycles; about 1/10th second in qemu.
```

#### ¿Cuánto dura un quantum en xv6-riscv?

El quantum en xv6-riscv dura 1000000 ciclos, lo que equivale aproximadamente a 1/10 de segundo en QEMU. Este valor se usa para configurar el registro CLINT_MTIMECMP y determinar el…