# Informe: Laboratorio 3-Planificador de procesos

Integrantes del grupo(03):
* Lautaro Ezequiel Deco
* Nehuen Emanuel Guevara
* Tomas Fabian Torres
* Fefe xd

## Primera parte: Estudiando el planificador de xv6-riscv


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



### ¿En qué parte del código ocurre el cambio de contexto en `xv6-riscv`? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?
...
### ¿El cambio de contexto consume tiempo de un *quantum*?
...

## Segunda parte: Medir operaciones de cómputo y de entrada/salida

Primero que nada definimos la **metrica** que utilizamos para esta parte del laboratorio y que usamos al hacer las mediciones de los experimentos. Notar que la naturaleza de las operaciones CPU e I/O es diferente por lo cual buscamos una metrica que trate de unificar o que permita comparar los resultados obtenidos
`cpubench`:
Metrica ->  k-operaciones de  CPU por tick (**kops / ticks**)  -> `metric = (total_cpu_kops * 1000) / elapsed_ticks`
`iobench`:
Metrica -> operaciones I/O por tick (**iops / ticks**) -> `metric = (total_iops * 1000) / elapsed_ticks`

Algo a destacar y notar es que los **kops** y **iops** son multiplicados por 1000, esta fue una solucion que dimos ya que nos dimos cuenta que a menudo la metrica en `iobench` daba un numero cercano a 0(aproximadamente), y como sabemos que se redondea, notamos que perdiamos informacion asi que decidimos multiplicar por 1000 para dar un tipo de "rescalado" a nuestra metrica para asi no perder informacion, luego en`cpubench`tambien lo hicimos para dar un tipo de uniformidad a la hora de comparar resultados.

### Experimento 1: ¿Cómo son planificados los programas iobound y cpubound?

#### 1. Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).

#### 2. ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.

#### 3. ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

#### 4.¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

#### 5. ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?
