
## ¿Qué política de planificación utiliza `xv6-riscv` para elegir el próximo proceso a ejecutarse?


La politica que utiliza es Raund Robbin ejectutando una lista de proceso donde cada      se ejecuta en un tiempo fijo antes de ser interrumpido y ejecutar el sieguiente.



##   ¿Cúales son los estados en los que un proceso puede permanecer en xv6-riscv y qué los hace cambiar de estado?

     enum procstate { UNUSED, USED, SLEEPING, RUNNABLE, RUNNING, ZOMBIE };
    
-**UNUSED** : El proceso no esta en uso
-**USED** :  El proceso esta en en uso
-**SLEEPING** : El proceso esta durmiendo, Esta esperando que ocurra un evento
-**RUNNEABLE** :  El proceso está listo para ser ejecutado. (READY)
-**RUNNING** : El proceso está en ejecución.
-**ZOMMBIE**:  El proceso ha terminado y está esperando a que su padre recoja su estado.
