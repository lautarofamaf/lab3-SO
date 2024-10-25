# Informe: Laboratorio 3-Planificador de procesos

Integrantes del grupo(03):
* Lautaro Ezequiel Deco
* Nehuen Emanuel Guevara
* Tomas Fabian Torres
* Fefe xd

## Primera parte: Estudiando el planificador de xv6-riscv


### ¿Qué política de planificación utiliza `xv6-riscv` para elegir el próximo proceso a ejecutarse?

Xv6 implementa una política de planificación muy básica llamada *Round Robin* donde cada proceso se ejecuta por turnos, es decir, cada uno se ejecuta en un tiempo fijo antes de ser interrumpido y pasar a ejecutar el sieguiente. Obviamente el proceso puede no haber acabado su ejecución por lo cual se deberá guardar toda la información asociada a ese proceso antes de ejecutar el proximo. Gracias a la implementacion del scheduler de xv6 todos los procesos terminaran en algun momento de ejecutarse por completo.



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

El quantum en xv6-riscv dura 1000000 ciclos, lo que equivale aproximadamente a 1/10 de segundo en QEMU. Este valor se usa para configurar el registro CLINT_MTIMECMP y determinar el momento exacto en el cual deberia haber una timer interrupt para avisarle al proceso que su quantum termino y devolverle el control al sistema operativo.



### ¿En qué parte del código ocurre el cambio de contexto en `xv6-riscv`? ¿En qué funciones un proceso deja de ser ejecutado? ¿En qué funciones se elige el nuevo proceso a ejecutar?

## ¿En qué parte del código ocurre el cambio de contexto en `xv6-riscv`?

Existe una función llamada **swtch**  implementada  en el archivo swtch.S (código maquina) la cual es la encargada de hacer posible el cambio de contexto. 

    
   Luego otra función **Sched** es el encargado de realizar el cambio de contexto cuando un proceso ya no puede continuar ejecutándose. Lo que hace este swtch es cambiar el contexto del proceso al del scheduler

	
   Luego esta función es llamada en otras como **scheduler**  que es la que decide que proceso se va a ejecutar a continuación y llama a swtch para realizar el cambio de contexto del scheduler al del proceso a ejecutarse 







## ¿En qué funciones un proceso deja de ser ejecutado? 

Hay varias funciones donde un proceso deja de ser ejecutado, ya sea por un cambio de contexto o por un cambio de estado. Entre ellas estan:
 - Funcion **swtch**: Es la funcion en codigo maquina encargada de hacer el cambio de contexto 
 - Funcion **sleep**: Esta fucion manda a dormir un  poceso 
 - Funcion **yeild**: Cunado un proceso usa esta funcion, sede voluntariamente CPU
 - Funcio **sched**: Realiza un cambio de contexto cuando sea necesario llamando a swtch
 -  Funcion **exit** : Provoca tambien con la diferencia que sera de manera definitiva



## ¿En qué funciones se elige el nuevo proceso a ejecutar?

Dentro de la función 
```c 
void scheduler(void)		
```
 se encuentra un bucle infinito donde  se recorre un  arreglo de procesos para buscar el siguiente proceso que este en estado RUNNEABLE Cuando encuentra uno, realiza el cambio de contexto usando la función :
 ```c 
swtch(&c->context, &p->context); // swtch 
```


-Donde  (**p->contex**)  es el puntero al contexto del proceso nuevo , el que se ejecutara a continuación.

-Luego **(c->context)**  es un puntero al proceso actual, donde los registros serán guardados para luego utilizarlos mas adelante  

-Luego la funcion guarda el indice donde se quedo para seguir recorriendo desde alli.


Ademas en la Función 
```c 
void sched(void)		
```
se realiza toda la lógica para que el proceso actual derive CPU para que se ejecute el siguiente proceso encontrado por la función anterior 



...
### ¿El cambio de contexto consume tiempo de un *quantum*?

La repuesta que daremos a esta pregunta es sí, y nos basamos para responderla en un pequeño experimento en el cual consiste en disminuir el quantum a "tiempos" o ticks pequeñitos y ver si funciona o no el *programa*. Dado que ni siquiera se puede *inicializar la terminal* a disminuir el tick a 100, lo que concluimos es que para iniciar la terminal se tienen que realizar una cierta cantidad de procesos los cuales usan context-switch para moverse entre ellos y nunca se pueden concretar puesto que el contex-switch en sí consume el quantum que tienen para ejecutarse llevando que el *programa* nunca ejecute codigo sino más bien se un bucle de context-switches.





## Segunda parte: Medir operaciones de cómputo y de entrada/salida

Primero que nada definimos la **metrica** que utilizamos para esta parte del laboratorio y que usamos al hacer las mediciones de los experimentos. Notar que la naturaleza de las operaciones CPU e I/O es diferente por lo cual buscamos una metrica que trate de unificar o que permita comparar los resultados obtenidos
`cpubench`:
Metrica ->  k-operaciones de  CPU por tick (**kops / ticks**)  -> `metric = (total_cpu_kops * 1000) / elapsed_ticks`
`iobench`:
Metrica -> operaciones I/O por tick (**iops / ticks**) -> `metric = (total_iops * 1000) / elapsed_ticks`

Algo a destacar y notar es que los **kops** y **iops** son multiplicados por 1000, esta fue una solucion que dimos ya que nos dimos cuenta que a menudo la metrica en `iobench` daba un numero cercano a 0(aproximadamente), y como sabemos que se redondea, notamos que perdiamos informacion asi que decidimos multiplicar por 1000 para dar un tipo de "rescalado" a nuestra metrica para asi no perder informacion, luego en`cpubench`tambien lo hicimos para dar un tipo de uniformidad a la hora de comparar resultados.

### Experimento 1: ¿Cómo son planificados los programas iobound y cpubound?

#### Resultados de experimento 1:

#### iobench 10 &
| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 4  | [iobench] | Perfomance  | 3820   | 2041       | 268          |
| 4  | [iobench] | Perfomance  | 3953   | 2309       | 259          |
| 4  | [iobench] | Perfomance  | 3968   | 2569       | 258          |
| 4  | [iobench] | Perfomance  | 3968   | 2828       | 258          |
| 4  | [iobench] | Perfomance  | 3953   | 3087       | 259          |
| 4  | [iobench] | Perfomance  | 3953   | 3346       | 259          |
| 4  | [iobench] | Perfomance  | 3968   | 3606       | 258          |
| 4  | [iobench] | Perfomance  | 3953   | 3865       | 259          |
| 4  | [iobench] | Perfomance  | 3953   | 4125       | 259          |
| 4  | [iobench] | Perfomance  | 3968   | 4384       | 258          |






#### iobench 10 &; iobench 10 &; iobench 10 &
| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 8 | [iobench] | Perfomance | 3112 | 2284 | 329 |
| 8 | [iobench] | Perfomance | 5595 | 2615 | 183 |
| 8 | [iobench] | Perfomance | 4923 | 2799 | 208 |
| 8 | [iobench] | Perfomance | 4946 | 3007 | 207 |
| 8 | [iobench] | Perfomance | 4718 | 3215 | 217 |
| 8 | [iobench] | Perfomance | 7937 | 3434 | 129 |
| 8 | [iobench] | Perfomance | 3820 | 3566 | 268 |
| 8 | [iobench] | Perfomance | 4162 | 3834 | 246 |
| 8 | [iobench] | Perfomance | 4899 | 4083 | 209 |
| 8 | [iobench] | Perfomance | 4357 | 4292 | 235 |

| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 5  | [iobench] | Perfomance  | 4063   | 2283       | 252          |
| 5  | [iobench] | Perfomance  | 5333   | 2537       | 192          |
| 5  | [iobench] | Perfomance  | 3710   | 2731       | 276          |
| 5  | [iobench] | Perfomance  | 4432   | 3009       | 231          |
| 5  | [iobench] | Perfomance  | 4047   | 3240       | 253          |
| 5  | [iobench] | Perfomance  | 4511   | 3493       | 227          |
| 5  | [iobench] | Perfomance  | 5657   | 3721       | 181          |
| 5  | [iobench] | Perfomance  | 3084   | 3902       | 332          |
| 5  | [iobench] | Perfomance  | 4231   | 4236       | 242          |
| 5  | [iobench] | Perfomance  | 5505   | 4482       | 186          |


| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 7  | [iobench] | Perfomance  | 4612   | 2284       | 222          |
| 7  | [iobench] | Perfomance  | 3112   | 2506       | 329          |
| 7  | [iobench] | Perfomance  | 5120   | 2837       | 200          |
| 7  | [iobench] | Perfomance  | 3864   | 3039       | 265          |
| 7  | [iobench] | Perfomance  | 3555   | 3306       | 288          |
| 7  | [iobench] | Perfomance  | 5278   | 3595       | 194          |
| 7  | [iobench] | Perfomance  | 4284   | 3789       | 239          |
| 7  | [iobench] | Perfomance  | 4946   | 4028       | 207          |
| 7  | [iobench] | Perfomance  | 5094   | 4235       | 201          |
| 7  | [iobench] | Perfomance  | 5171   | 4436       | 198          |


#### cpubench 10 &

| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 12 | [cpubench] | Perfomance  | 3334360 | 13404      | 161          |
| 12 | [cpubench] | Perfomance  | 3334360 | 13566      | 161          |
| 12 | [cpubench] | Perfomance  | 3334360 | 13728      | 161          |
| 12 | [cpubench] | Perfomance  | 3313777 | 13889      | 162          |
| 12 | [cpubench] | Perfomance  | 3334360 | 14051      | 161          |
| 12 | [cpubench] | Perfomance  | 3334360 | 14213      | 161          |
| 12 | [cpubench] | Perfomance  | 3334360 | 14375      | 161          |
| 12 | [cpubench] | Perfomance  | 3313777 | 14536      | 162          |
| 12 | [cpubench] | Perfomance  | 3334360 | 14698      | 161          |
| 12 | [cpubench] | Perfomance  | 3334360 | 14860      | 161          |


#### cpubench 10 &; cpubench 10 &; cpubench 10 &

| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 22 | [cpubench] | Perfomance  | 1111453 | 21567      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 22053      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 22539      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 23025      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 23511      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 23994      | 483          |
| 22 | [cpubench] | Perfomance  | 1111453 | 24480      | 483          |
| 22 | [cpubench] | Perfomance  | 1118400 | 24966      | 480          |
| 22 | [cpubench] | Perfomance  | 1111453 | 25449      | 483          |
| 22 | [cpubench] | Perfomance  | 1156965 | 25932      | 464          |


| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 21 | [cpubench] | Perfomance  | 1118400 | 21566      | 480          |
| 21 | [cpubench] | Perfomance  | 1125433 | 22049      | 477          |
| 21 | [cpubench] | Perfomance  | 1118400 | 22529      | 480          |
| 21 | [cpubench] | Perfomance  | 1118400 | 23009      | 480          |
| 21 | [cpubench] | Perfomance  | 1125433 | 23492      | 477          |
| 21 | [cpubench] | Perfomance  | 1125433 | 23972      | 477          |
| 21 | [cpubench] | Perfomance  | 1125433 | 24452      | 477          |
| 21 | [cpubench] | Perfomance  | 1118400 | 24932      | 480          |
| 21 | [cpubench] | Perfomance  | 1118400 | 25412      | 480          |
| 21 | [cpubench] | Perfomance  | 1125433 | 25895      | 477          |



| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 19 | [cpubench] | Perfomance  | 1118400 | 21568      | 480          |
| 19 | [cpubench] | Perfomance  | 1111453 | 22048      | 483          |
| 19 | [cpubench] | Perfomance  | 1111453 | 22531      | 483          |
| 19 | [cpubench] | Perfomance  | 1118400 | 23017      | 480          |
| 19 | [cpubench] | Perfomance  | 1111453 | 23497      | 483          |
| 19 | [cpubench] | Perfomance  | 1118400 | 23980      | 480          |
| 19 | [cpubench] | Perfomance  | 1118400 | 24463      | 480          |
| 19 | [cpubench] | Perfomance  | 1118400 | 24946      | 480          |
| 19 | [cpubench] | Perfomance  | 1118400 | 25429      | 480          |
| 19 | [cpubench] | Perfomance  | 1132556 | 25912      | 474          |


#### iobench 10 &; cpubench 10 &; cpubench 10 &; cpubench 10 &

| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 23 | [cpubench] | Perfomance  | 1034358 | 30860      | 519          |
| 23 | [cpubench] | Perfomance  | 1034358 | 31382      | 519          |
| 23 | [cpubench] | Perfomance  | 1040372 | 31901      | 516          |
| 23 | [cpubench] | Perfomance  | 1040372 | 32420      | 516          |
| 23 | [cpubench] | Perfomance  | 1034358 | 32939      | 519          |
| 23 | [cpubench] | Perfomance  | 1040372 | 33461      | 516          |
| 23 | [cpubench] | Perfomance  | 1034358 | 33977      | 519          |
| 23 | [cpubench] | Perfomance  | 1034358 | 34499      | 519          |
| 23 | [cpubench] | Perfomance  | 1034358 | 35018      | 519          |
| 23 | [cpubench] | Perfomance  | 1883621 | 35540      | 285          |


| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 26 | [cpubench] | Perfomance  | 1100065 | 30866      | 488          |
| 26 | [cpubench] | Perfomance  | 1111453 | 31354      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 31840      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 32326      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 32809      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 33295      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 33781      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 34267      | 483          |
| 26 | [cpubench] | Perfomance  | 1111453 | 34753      | 483          |
| 26 | [cpubench] | Perfomance  | 1134951 | 35239      | 473          |


| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 25 | [cpubench] | Perfomance  | 1113759 | 30862      | 482          |
| 25 | [cpubench] | Perfomance  | 1118400 | 31347      | 480          |
| 25 | [cpubench] | Perfomance  | 1118400 | 31830      | 480          |
| 25 | [cpubench] | Perfomance  | 1111453 | 32310      | 483          |
| 25 | [cpubench] | Perfomance  | 1118400 | 32796      | 480          |
| 25 | [cpubench] | Perfomance  | 1118400 | 33279      | 480          |
| 25 | [cpubench] | Perfomance  | 1118400 | 33762      | 480          |
| 25 | [cpubench] | Perfomance  | 1111453 | 34242      | 483          |
| 25 | [cpubench] | Perfomance  | 1118400 | 34725      | 480          |
| 25 | [cpubench] | Perfomance  | 1118400 | 35208      | 480          |

| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 21 | [iobench] | Perfomance  | 201    | 30879      | 5082         |
| 21 | [iobench] | Perfomance  | 3968   | 35961      | 258          |
| 21 | [iobench] | Perfomance  | 3953   | 36220      | 259          |
| 21 | [iobench] | Perfomance  | 3953   | 36479      | 259          |
| 21 | [iobench] | Perfomance  | 3938   | 36738      | 260          |
| 21 | [iobench] | Perfomance  | 3968   | 36998      | 258          |
| 21 | [iobench] | Perfomance  | 3953   | 37257      | 259          |
| 21 | [iobench] | Perfomance  | 3968   | 37516      | 258          |
| 21 | [iobench] | Perfomance  | 3953   | 37775      | 259          |
| 21 | [iobench] | Perfomance  | 3984   | 38035      | 257          |

#### cpubench 10 &; iobench 10 &; iobench 10 &; iobench 10 &

| id | Type       | name_metric | Metric  | Start_tick | Elapsed_tick |
|----|------------|-------------|---------|------------|--------------|
| 5  | [cpubench] | Perfomance  | 2949626 | 3158       | 182          |
| 5  | [cpubench] | Perfomance  | 2982400 | 3341       | 180          |
| 5  | [cpubench] | Perfomance  | 2965922 | 3522       | 181          |
| 5  | [cpubench] | Perfomance  | 2982400 | 3704       | 180          |
| 5  | [cpubench] | Perfomance  | 2949626 | 3884       | 182          |
| 5  | [cpubench] | Perfomance  | 2670805 | 4067       | 201          |
| 5  | [cpubench] | Perfomance  | 2901794 | 4268       | 185          |
| 5  | [cpubench] | Perfomance  | 2933508 | 4454       | 183          |
| 5  | [cpubench] | Perfomance  | 2982400 | 4638       | 180          |
| 5  | [cpubench] | Perfomance  | 2508560 | 4818       | 214          |



| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 10 | [iobench] | Perfomance  | 522    | 3162       | 1959         |
| 10 | [iobench] | Perfomance  | 4063   | 5121       | 252          |
| 10 | [iobench] | Perfomance  | 3002   | 5373       | 341          |
| 10 | [iobench] | Perfomance  | 3723   | 5714       | 275          |
| 10 | [iobench] | Perfomance  | 4196   | 5991       | 244          |
| 10 | [iobench] | Perfomance  | 4899   | 6235       | 209          |
| 10 | [iobench] | Perfomance  | 4432   | 6444       | 231          |
| 10 | [iobench] | Perfomance  | 7111   | 6677       | 144          |
| 10 | [iobench] | Perfomance  | 5278   | 6824       | 194          |
| 10 | [iobench] | Perfomance  | 3953   | 7019       | 259          |


| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 9  | [iobench] | Perfomance  | 522    | 3162       | 1958         |
| 9  | [iobench] | Perfomance  | 4718   | 5120       | 217          |
| 9  | [iobench] | Perfomance  | 5361   | 5339       | 191          |
| 9  | [iobench] | Perfomance  | 4452   | 5530       | 230          |
| 9  | [iobench] | Perfomance  | 5278   | 5760       | 194          |
| 9  | [iobench] | Perfomance  | 3923   | 5954       | 261          |
| 9  | [iobench] | Perfomance  | 5851   | 6215       | 175          |
| 9  | [iobench] | Perfomance  | 3631   | 6393       | 282          |
| 9  | [iobench] | Perfomance  | 5720   | 6675       | 179          |
| 9  | [iobench] | Perfomance  | 7474   | 6854       | 137          |



| id | Type      | name_metric | Metric | Start_tick | Elapsed_tick |
|----|-----------|-------------|--------|------------|--------------|
| 7  | [iobench] | Perfomance  | 1025   | 3159       | 999          |
| 7  | [iobench] | Perfomance  | 1199   | 4158       | 854          |
| 7  | [iobench] | Perfomance  | 4762   | 5012       | 215          |
| 7  | [iobench] | Perfomance  | 4633   | 5227       | 221          |
| 7  | [iobench] | Perfomance  | 4357   | 5448       | 235          |
| 7  | [iobench] | Perfomance  | 3792   | 5683       | 270          |
| 7  | [iobench] | Perfomance  | 5953   | 5955       | 172          |
| 7  | [iobench] | Perfomance  | 5171   | 6128       | 198          |
| 7  | [iobench] | Perfomance  | 3631   | 6326       | 282          |
| 7  | [iobench] | Perfomance  | 3112   | 6609       | 329          |




#### 1. Describa los parámetros de los programas cpubench e iobench para este experimento (o sea, los define al principio y el valor de N. Tener en cuenta que podrían cambiar en experimentos futuros, pero que si lo hacen los resultados ya no serán comparables).

#### 2. ¿Los procesos se ejecutan en paralelo? ¿En promedio, qué proceso o procesos se ejecutan primero? Hacer una observación cualitativa.

#### 3. ¿Cambia el rendimiento de los procesos iobound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

#### 4.¿Cambia el rendimiento de los procesos cpubound con respecto a la cantidad y tipo de procesos que se estén ejecutando en paralelo? ¿Por qué?

#### 5. ¿Es adecuado comparar la cantidad de operaciones de cpu con la cantidad de operaciones iobound?