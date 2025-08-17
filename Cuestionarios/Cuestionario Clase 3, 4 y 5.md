*1- a) Explique la semántica de un semáforo. Diferencie el caso de los semáforos generales y los binarios.     b) Indique los posibles valores finales de x en el siguiente programa (justifique claramente su* 
      *respuesta):* 
	  int x = 4; 
	  sem s1 = 1, s2 = 0; 
	  co 
		P(s1); x = x * x ; V(s1); // P(s2); P(s1); x = x * 3; V(s1): // P(s1); x = x - 2; V(s2); V(s1); 
	  oc 
- a) 
	- P(s) < await(s>0) s=s-1 > | V(s) < s=s+1 > --> semáforos generales, la operación V no es bloqueante.
	- P(s) < await(s>0) s=s-1 > | V(s) < await(s<1) s=s+1 > --> Los sem binarios limitan la cota superior. Ambas operaciones son bloqueantes.
- b) El proceso 2 nunca va a poder suceder antes que el 3, ya que requiere hacer un P del semáforo s2.
	- 42 sucede primero 1, luego 3 y por último 2.
	- 36 sucede primero 3, luego 2 y por último 1.
	- 12 sucede primero 3, luego 1 y por último 2.

*2- Desarrolle utilizando semáforos una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes).* 
- Es un problema que requiere exclusión mutua selectiva para evitar deadlock (o toma todo o nada).
```c
	sem pedido = 0; sem mutex = 1; array tenedores[9] = ([9]true); cola colaF; sem mutexCola = 1;
	Process Administrador {
	 int aux;
	 while(true){
	  P(pedido);
	  P(mutexCola);
	  pop(colaF,aux);
	  V(mutexCola);
	  P(mutex);
	  otorgarTenedores(tenedores,aux); //función que pone en false los tenedores necesarios.
	  V(mutex);
	  V(comer[aux]);
	 }
	}
	Process Filósofo [id: 0..4]{
	 while(true){
	  P(mutexCola);
	  push(colaF,id);
	  V(mutexCola);
	  V(pedido);
	  P(comer[id]);
	  P(mutex);
	  devolverTenedores(tenedores,id); //función que pone en true los tenedores necesarios.
	  V(mutex);
	 }
	}
```

*3- Describa la técnica de Passing the Baton. ¿Cuál es su utilidad en la resolución de problemas mediante semáforos?* 
- Técnica general para implementar sentencias await. Cuando un proceso está dentro de una sección crítica tiene el baton (permiso de ejecutar). Cuando sale de su sección crítica pasa el baton a otro proceso, si no hay nadie esperando el baton se libera para ser tomado por cualquier proceso que llegue.
- La técnica Passing the Baton es un patrón de sincronización utilizado en la programación concurrente, específicamente con semáforos, para resolver el problema de la inanición y garantizar que cada proceso obtenga su turno para acceder a un recurso compartido (siguiendo algún orden particular, por ejemplo de llegada).

*4- Modifique las soluciones de Lectores-Escritores con semáforos de modo de no permitir más de 10 lectores simultáneos en la BD y además que no se admita el ingreso a más lectores cuando hay escritores esperando.* 
```c
	sem mutex = 1; sem lectores[m] = ([m]0); sem escritores[n] = ([n]0)
	int cantL = 0; int cantE = 0; int cant = 0; cola colaE; cola colaL;
	Process Escritor[id:0..N]{
	 while (true){
	  P(mutex);
	  if(cant > 0){
	   cantE = cantE + 1;
	   push(colaE,id);
	   V(mutex);
	   P(escritores[id]);
	   P(mutex);
	   cantE = cantE - 1;
	   V(mutex);
	  }
	  else{
	   cant = cant + 1;
	   V(mutex)
	  }
	  //sección crítica;
	  P(mutex);
	  if(cantE > 0){
	   V(escritores[pop(colaE,id)]);
	  }
	  elif(cantL > 0){
	   V(lectores[pop(colaL,id)]);
	  }
	  else{
	   cant = cant - 1;
	  }
	  V(mutex);
	 }
	 }
	}
	Process Lector[id:0..M]{
	 while(true){
	  P(mutex);
	  if(cant == 10)or(cantE > 0) {
	   cantL = cantL + 1;
	   push(colaL,id);
	   V(mutex);
	   P(lectores[id]);
	   P(mutex);
	   cantL = cantL - 1;
	   V(mutex);
	  }
	  else{
	   cant = cant + 1;
	   V(mutex)
	  }
	  //sección crítica;
	  P(mutex);
	  if(cantE > 0){
	   V(escritores[pop(colaE,id)]);
	  }
	  elif(cantL > 0){
	   V(lectores[pop(colaL,id)]);
	  }
	  else{
	   cant = cant - 1;
	  }
	  V(mutex);
	 }
	}
```

*5- Describa el funcionamiento de los monitores como herramienta de sincronización.* 
- Módulos de programa con más estructura. Mecanismo de abstracción de datos (tipo de dato abstracto), encapsulan las representaciones de recursos y brindan un conjunto de operaciones que son los únicos medios para manipular los recursos. Contiene variables que almacenan el del recurso y procedimientos que implementan las operaciones sobre él. Procesos activos y monitores pasivos.
- Poseen una interfaz(procedimientos) y un cuerpo(conjunto de variables que representan el recurso). Operaciones: Wait(se duerme al final de la cola y libera el monitor), Signal(despierta el primer proceso de la cola), Signal_All(despierta todos los procesos).
- La exclusión mutua está implícita asegurando que los procedures en el mismo monitor no se ejecutan concurrentemente y la sincronización por condición se trabaja mediante variables condición de manera explícita.

*6- ¿Qué diferencias existen entre las disciplinas de señalización “Signal and wait” y “Signal and continue”?* 
- Signal and Wait: el proceso que hace el signal pasa a competir por el monitor con el resto de los procesos y el proceso despertado automáticamente adquiere le monitor.
- Signal and Continue: el proceso que despierta al proceso dormido primero termina de ejecutar el procedimiento o se duerme, luego ambos procesos pasan a competir por el monitor.

*7- ¿En qué consiste la técnica de Passing the Condition y cuál es su utilidad en la resolución de problemas con monitores? ¿Qué relación encuentra entre passing the condition y passing the baton?*
- Los procesos que no pueden continuar su ejecución porque una condición no se cumple (por ejemplo, un búfer está vacío o lleno) se bloquean en una variable de condición. La técnica de Passing the Condition introduce una regla estricta para la liberación de la exclusión mutua, de manera tal que el proceso que está saliendo del monitor cede inmediatamente la exclusión mutua (el monitor) al proceso que acaba de despertar.
- La principal utilidad de Passing the Condition es la eliminación de la inanición y la mejora de la eficiencia y predictibilidad del código concurrente.
- Ambas técnicas, Passing the Condition y Passing the Baton, tienen un objetivo común: garantizar una transferencia justa y ordenada del control entre procesos para evitar la inanición, solo que una se implementa con monitores y la otra con semáforos (esta última requiere controlar explícitamente la exclusión mutua del recurso compartido, haciendo su implementación un poco más compleja)

8- Desarrolle utilizando monitores una solución centralizada al problema de los filósofos, con un administrador único de los tenedores, y posiciones libres para los filósofos (es decir, cada filósofo puede comer en cualquier posición siempre que tenga los dos tenedores correspondientes). 

*9- Sea la siguiente solución propuesta al problema de alocación SJN:* 
monitor SJN { 
  bool libre = true; cond turno; 
  procedure request(int tiempo) { 
   if (not libre) wait(turno, tiempo); 
   libre = false; 
  } 
  procedure release() { 
   libre = true 
   signal(turno); 
  } 
} 
*a) Funciona correctamente con disciplina de señalización Signal and Continue?* 
*b) Funciona correctamente con disciplina de señalización Signal and Wait? Justifique claramente sus respuestas.*
- No funciona correctamente con signal and continue ya que pone a competir al proceso despertado con los demás, por lo que no hay garantía de que se respete el orden.
- Funciona, ya que el proceso despertado toma directamente el monitor y pone automáticamente libre en false, garantizando la ocupación del recurso.

*10- Modifique la solución anterior para el caso de no contar con una instrucción wait con prioridad.* 
monitor SJN { 
  bool libre = true; cond turno[n]; cola colaE; int id;
  procedure request(int tiempo, int id) { 
   if (not libre) {
    insertarOrdenado(cola, id, tiempo);
    wait(turno[id]); 
   }
   libre = false; 
  } 
  procedure release() { 
   libre = true; 
   signal(turno[pop(colaE, id)]); 
  } 
} 

*11- Modifique utilizando monitores las soluciones de Lectores-Escritores visto en la teoría, de modo de no permitir más de 10 lectores simultáneos en la BD, y además que no se admita el ingreso a más lectores cuando hay escritores esperando para ingresar.* 
```c
monitor Controlador_RW
{
  int nr = 0, nw = 0, dr = 0, dw = 0;
  cond ok_leer, ok_escribir;

  procedure pedido_leer() {
    if (nw > 0 OR dw > 0 OR nr >= 10) {
      dr = dr + 1;
      wait(ok_leer);
    }
    nr = nr + 1;
  }

  procedure libera_leer() {
    nr = nr - 1;
    if (nr == 0 and dw > 0) {
      dw = dw - 1;
      signal(ok_escribir);
    }
    else if (dr > 0) { 
	  dr = dr - 1;
	  signal(ok_leer);
    }
  }

  procedure pedido_escribir() {
    if (nr > 0 OR nw > 0) {
      dw = dw + 1;
      wait(ok_escribir);
    }
    nw = nw + 1;
  }

  procedure libera_escribir() {
    nw = nw - 1;
    if (dw > 0) {
      dw = dw - 1;
      signal(ok_escribir);
    }
    else if (dr > 0) {
      dr = dr - 1;
	  signal(ok_leer);
    }
  }
}
```

*12- Resuelva con monitores el siguiente problema: Tres clases de procesos comparten el acceso a una lista enlazada: searchers, inserters y deleters. Los searchers sólo examinan la lista, y por lo tanto pueden ejecutar concurrentemente unos con otros. Los inserters agregan nuevos ítems al final de la lista; las inserciones deben ser mutuamente exclusivas para evitar insertar dos ítems casi al mismo tiempo. Sin embargo, un insert puede hacerse en paralelo con uno o más searches. Por último, los deleters remueven ítems de cualquier lugar de la lista. A lo sumo un deleter puede acceder la lista a la vez, y el borrado también debe ser mutuamente exclusivo con searches e inserciones.* 
- Hecho con Gemini, no lo voy a revisar.
```c
monitor ListaMonitor {
    int num_searchers = 0;
    int num_inserters = 0;
    int num_deleters = 0;
    Condition ok_search, ok_insert, ok_delete;

    procedure pedido_search() {
        if (num_deleters > 0 || !ok_delete.empty()) {
            ok_search.wait();
        }
        num_searchers++;
    }

    procedure libera_search() {
        num_searchers--;
        if (num_searchers == 0) {
            if (!ok_delete.empty()) {
                ok_delete.signal();
            }
            else if (!ok_insert.empty()) {
                ok_insert.signal();
            }
            else {
                ok_search.signalAll();
            }
        }
    }

    procedure pedido_insert() {
        if (num_inserters > 0 || num_deleters > 0 || !ok_delete.empty()) {
            ok_insert.wait();
        }
        num_inserters++;
    }

    procedure libera_insert() {
        num_inserters--;
        if (!ok_delete.empty()) {
            ok_delete.signal();
        }
        else if (!ok_insert.empty()) {
            ok_insert.signal();
        }
        else {
            ok_search.signalAll();
        }
    }

    procedure pedido_delete() {
        if (num_searchers > 0 || num_inserters > 0 || num_deleters > 0) {
            ok_delete.wait();
        }
        num_deleters++;
    }

    procedure libera_delete() {
        num_deleters--;
        if (!ok_delete.empty()) {
            ok_delete.signal();
        }
        else if (!ok_insert.empty()) {
            ok_insert.signal();
        }
        else {
            ok_search.signalAll();
        }
    }
}

Process Searcher {
  while(true) {
    ListaMonitor.pedido_search();
    // ...acceso a la lista
    ListaMonitor.libera_search();
  }
}

Process Inserter {
  while(true) {
    ListaMonitor.pedido_insert();
    // ...acceso a la lista
    ListaMonitor.libera_insert();
  }
}

Process Deleter {
  while(true) {
    ListaMonitor.pedido_delete();
    // ...acceso a la lista
    ListaMonitor.libera_delete();
  }
}
```

*13- El problema del “Puente de una sola vía” (One-Lane Bridge): autos que provienen del Norte y del Sur llegan a un puente con una sola vía. Los autos en la misma dirección pueden atravesar el puente al mismo tiempo, pero no puede haber autos en distintas direcciones sobre el puente.* 
*a) Desarrolle una solución al problema, modelizando los autos como procesos y sincronizando con un monitor (no es necesario que la solución sea fair ni dar preferencia a ningún tipo de auto).* 
*b) Modifique la solución para asegurar fairness (Pista: los autos podrían obtener turnos).*
- b) Hecho con Gemini, no lo voy a revisar.
```c
monitor ControladorPuente {
    int pasando_norte = 0, pasando_sur = 0;
    int espera_norte = 0, espera_sur = 0;
    int turno = 0; // 0 para Norte, 1 para Sur

    Condition ok_norte, ok_sur;

    procedure entra_norte() {
        if (pasando_sur > 0 || (pasando_norte == 0 && turno == 1)) {
            espera_norte++;
            wait(ok_norte);
            espera_norte--;
        }
        pasando_norte++;
    }

    procedure sale_norte() {
        pasando_norte--;
        if (pasando_norte == 0) {
            if (espera_sur > 0) {
                turno = 1; // Cede el turno al Sur
                signal(ok_sur);
            } 
            else if (espera_norte > 0) {
                turno = 0; // Mantiene el turno para el Norte
                signal(ok_norte);
            }
        }
    }

    procedure entra_sur() {
        if (pasando_norte > 0 || (pasando_sur == 0 && turno == 0)) {
            espera_sur++;
            wait(ok_sur);
            espera_sur--;
        }
        pasando_sur++;
    }

    procedure sale_sur() {
        pasando_sur--;
        if (pasando_sur == 0) {
            if (espera_norte > 0) {
                turno = 0; // Cede el turno al Norte
                signal(ok_norte);
            } 
            else if (espera_sur > 0) {
                turno = 1; // Mantiene el turno para el Sur
                signal(ok_sur);
            }
        }
    }
}
```

*14- Indicar las características generales de Pthreads. Explicar cómo se maneja la sincronización por exclusión mutua y por condición en Pthreads. Indicar la relación entre ambas. Explicar cómo se pueden simular los monitores en Pthreads.*
- Pthreads es una biblioteca en C para crear threads(procesos livianos con su propio PC y pila, pero no controlan el contexto pesado, por ej, tablas de página), asignarles atributos, terminarlos, identificarlos, etc.
- Proveen funciones reentrantes, las cuales pueden ser llamadas en simultáneo por varios procesos ya que no trabajan con variables locales ni estáticas. Además, provee funciones para crear/create (genera el hilo y manda a ejecutar la función pasada por parámetro con los argumentos que también fueron pasados por parámetro), exit (para devolver algún valor al final del create), join (para esperar a que algún hilo en particular haya terminado su ejecución, lo recibe por parámetro el hilo buscado), cancel (solicitar la cancelación de un hilo pasado por parámetro). 
- Exclusión mutua, se implementa usando mutex locks por medio de variables mutex (estados locked/unlocked), sólo un thread puede bloquear un mutex. Todos los mutex se inicializan automáticamente como unlocked. Pthreads soporta 3 tipos de mutex: Normal, Recursive, Error Check.
	- Normal: no permite bloqueos recursivos, una vez bloqueado solo se puede desbloquear, sino se genera deadlock.
	- Recursive: permite bloqueos recursivos, puede volver a bloquearse un mutex siempre y cuando sea el mismo thread/proceso (lleva un contador de bloqueos).
	- Error Check: no permite bloqueos recursivos, pero no bloquea los procesos que lo intentan, sino que informa un error.
- Variables condición, se utiliza para bloquear un thread hasta que se cumpla una condición. Cuando se cumple la condición, se alerta a los procesos que se encontraban esperando. Todas las variables condición tienen un mutex asociado, cada thread bloquea el mutex y consulta el predicado. El mutex protege la variable o el estado del sistema que se está evaluando como la "condición". Esto evita que dos hilos evalúen o modifiquen la condición al mismo tiempo.
- Pthreads no posee monitores, hay que simularlos. Con variables mutex se logra la exclusión mutua propia del monitor y con las variables condición la sincronización. Se definen funciones que representan los procedimientos del monitor. Cada uno de estos procedimientos debe: 
	- Comenzar con un pthread_mutex_lock() para adquirir la exclusión mutua. 
	- Contener la lógica del procedimiento, incluyendo las llamadas a pthread_cond_wait() si es necesario. 
	- Finalizar con un pthread_mutex_unlock() para liberar la exclusión mutua.