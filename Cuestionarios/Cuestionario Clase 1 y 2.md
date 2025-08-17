*1- Mencione al menos 3 ejemplos donde pueda encontrarse concurrencia* 
- Navegador web
- Acceso al disco mientras otras aplicaciones siguen funcionando.
- Varios usuarios conectados a un mismo sistema.

*2- Escriba una definición de concurrencia. Diferencie procesamiento secuencial, concurrente y paralelo.* 
- La concurrencia es el concepto basado en ejecutar tareas de manera simultánea, pero este concepto se basa en el software y requiere comunicación y sincronización. Mejora tiempos de ejecución, utiliza mejor la CPU y la mayoría de los problemas reales son inherentemente concurrentes. 
- Los distintos procesamientos difieren en la manera de resolver un problema, el tiempo que se utiliza e incluso el paradigma a usar. 
	- EL procesamiento secuencial toma el problema y lo resuelve paso por paso esperando resolver el primero para continuar con el próximo paso. Un único hilo de ejecución.
	- El procesamiento concurrente realiza estos pasos de manera no determinística intercalando estos pasos. Múltiples hilos de ejecución.
	- Por último, el procesamiento paralelo es el procesamiento concurrente pero ayudado por el hardware, permitiendo que estas tareas se puedan ejecutar literalmente en simultáneo. Múltiples hilos de ejecución.

*3- Describa el concepto de deadlock y qué condiciones deben darse para que ocurra.* 
- Cuando 2 o más procesos se quedan esperando recursos que posee el otro y se bloquean mutuamente (A espera recurso que tiene B y B espera recurso que tiene A).
- 4 propiedades necesarias y suficientes para que exista deadlock:
	- Recursos reusables serialmente: los procesos comparten recursos que pueden usar con exclusión mutua.
	- Adquisición incremental: los procesos mantienen recursos mientras esperan el resto.
	- No-preemption: una vez adquiridos, los recursos no pueden quitarse, sólo liberarse voluntariamente.
	- Espera cíclica: existe una cadena circular de procesos que requieren recursos (A tiene recurso que requiere B, B de C, C de D y D de A).
- Con asegurarse que una de las propiedades no se cumpla ya alcanza para evitar el deadlock.

*4- Defina inanición. Ejemplifique.*
- Que un proceso nunca logre acceder a un recurso, queda en espera constante, ocurre cuando hay un mal manejo de recursos compartidos. No se logra la propiedad Fairness.
- Ej: Muchas personas queriendo acceder a un recurso por orden alfabético. Las personas con Z probablemente nunca puedan acceder al recurso.

*5- ¿Qué entiende por no determinismo? ¿Cómo se aplica este concepto a la ejecución concurrente?*
- Significa que dos ejecuciones de un mismo programa no necesariamente son iguales. Se da gracias a la competencia de procesos donde el momento de acceso a las secciones críticas puede diferir.

*6- Defina comunicación. Explique los mecanismos de comunicación que conozca.* 
- Es el modo en el que los procesos se organizan y transmiten datos entre tareas concurrentes.
- Memoria Compartida: los procesos toman datos de un mismo espacio de direcciones, actúan de manera coordinada sobre este espacio, para ello deben bloquear y liberar el recurso compartido (el acceso a la memoria). --> Semáforos y monitores
- Pasaje de Mensajes: es necesario una red de interconexión, establecer un canal lógico o físico. Los procesos deben saber cuando deben transmitir y cuando leer mensajes. --> PMA, PMS y Rendezvous

*7-  a) Defina sincronización. Explique los mecanismos de sincronización que conozca.* 
	*b) ¿En un programa concurrente pueden estar presentes más de un mecanismo de sincronización? En caso afirmativo, ejemplifique* 
- Es la posesión de información acerca de otro proceso para coordinar actividades.
- Exclusión Mutua: un solo proceso puede usar el recurso compartido, solo uno accede a la sección crítica del programa.
- Por Condición: bloquea la ejecución de un proceso hasta que se cumpla una condición.
- Sí, Ej: buffer limitado con productores y consumidores. En este caso debe tenerse en cuenta que la cola o la estructura utilizada no esté llena (condición), además deben acceder a la cola de a uno (exclusión mutua).

*8- ¿Qué significa el problema de “interferencia” en programación concurrente? ¿Cómo puede evitarse?*
- Interferencia: un proceso toma una acción que invalida las suposiciones hechas por otro proceso.
- Ej: controla que no sea 0 la variable que divide otro valor pero cuando va a hacer la división otro proceso modifica la variable y la pone en 0, entonces falla el programa. Creo que la solución podría ser incluir todo en una única zona crítica y realizar exclusión mutua. Por lo menos para este ejemplo sirve, controlaría la condición y dividiría el número como una única acción atómica.
- Solo pasa en memoria compartida, no en pasaje de mensaje porque no comparten memoria.

*9- ¿En qué consiste la propiedad de “A lo sumo una vez” y qué efecto tiene sobre las sentencias de un programa concurrente? De ejemplos de sentencias que cumplan y de sentencias que no cumplan con ASV.* 
- Una sentencia de asignación x = e cumple con la propiedad de "a lo sumo una vez" si:
	- e contiene a lo sumo una referencia crítica y x no es referenciada por otro proceso.
	- e no contiene referencias críticas, en cuyo caso x puede ser leída por otro proceso.
	- O sea puede haber a lo sumo una variable compartida y puede ser referenciada a lo sumo una vez.
	- Ej:
		- int x=0, y=0; co x=x+1 // y=y+1 oc; no hay referencias críticas. x = 1 e y = 1
		- int x=0, y=0; co x=y+1 // y=y+1 oc; solo hay una referencia crítica en el primer proceso. y = 1 y x = 1/2
		- int x=0, y=0; co x=y+1 // y=x+1 oc; no satisface ASV, hay 2 referencias críticas que podrían causar x = 1 e y = 1 (mal)

10- Dado el siguiente programa concurrente: 
	x = 2; y = 4; z = 3;
	co 
	x = y - z // z = x * 2 // y = y - 1 
	oc
a) ¿Cuáles de las asignaciones dentro de la sentencia co cumplen con ASV?. Justifique claramente. 
b) Indique los resultados posibles de la ejecución 
	Nota 1: las instrucciones NO SON atómicas. 
	Nota 2: no es necesario que liste TODOS los resultados, pero si los que sean representativos de las diferentes situaciones que pueden darse. 
- a) El primer proceso no cumple.
- b) x = y - z
		- cargar y en un registro (1.1)
		- restarle z (1.2)
		- guardar resultado en x (1.3)
	z = x * 2 
		- cargar x en un registro (2.1)
		- multiplicar por 2 (2.2)
		- guardar en z (2.3)
	y = y - 1
		- cargar y en un registro (3.1)
		- restarle 1 (3.2)
		- guardar el resultado en y (3.3)
	- Resultados posibles:
		- 1.1 1.2 1.3 2.1 2.2 2.3 3.1 3.2 3.3 = x = 1 ; z = 2 ; y = 3
		- 1.1 2.1 3.1 1.2 2.2 3.2 1.3 2.3 3.3 = x = 0 ; z = 4 ; y = 3
		- 1.1 1.2 2.1 2.2 3.1 3.2 1.3 2.3 3.3 = x = 1 ; z = 2 ; y = 3
		- 1.1 2.1 2.2 2.3 1.2 1.3 3.1 3.2 3.3 = x = 1 ; z = 2 ; y = 3
		- 1.1 2.1 2.2 2.3 3.1 3.2 3.3 1.2 1.3 = x = 1 ; z = 2 ; y = 3

*11- Defina acciones atómicas condicionales e incondicionales. Ejemplifique.* 
- < e > --> indica que la expresión debe ser evaluada atómicamente (o sea, el estado durante la expresión es invisible para el resto y se ejecuta como una acción de grano grueso). O sea sirve para exclusión mutua. Ej: < x = x+1; y = y+1 >
- < await (B) s; > --> igual que el anterior solo que agrega una condición y espera a que se cumpla para ejecutarse (se demora). Ej: < await (s > 0) s = s-1>

*12- Defina propiedad de seguridad y propiedad de vida.* 
- Seguridad (safety): nada malo le ocurre a un proceso, asegura estados consistentes. Una falla de seguridad indica que algo anda mal. Ej: exclusión mutua, ausencia de interferencia entre procesos, partial correctness. 
- Vida (liveness): eventualmente ocurre algo bueno con una actividad, progresa (no hay deadlocks). Ej: terminación, dependen de las políticas de scheduling.

*13- ¿Qué es una política de scheduling? Relacione con fairness. ¿Qué tipos de fairness conoce? ¿Por qué las propiedades de vida dependen de la política de scheduling?* 
- Fairness: trata de garantizar que todos los procesos tengan chance de avanzar.
- Esto está estrechamente relacionado con las políticas de scheduling ya que son las que determinan cual será la próxima acción atómica elegible a ejecutarse. Entonces la política a utilizarse debe garantizar que todos los procesos accedan al procesador.
- Tipos de Fairness:
	- Fairness Incondicional: cuando toda acción atómica incondicional que es elegible se ejecuta eventualmente. Ej: Round Robin en monoprocesador y Paralelismo en multiprocesador.
	- Fairness Débil: cuando cumple Fairness Incondicional y toda acción atómica condicional que se vuelve elegible se ejecuta eventualmente (asumiendo que se vuelve "true" y se conserva hasta ser elegida). Ej: Round Robin.
	- Fairness Fuerte: cuando cumple Fairness Incondicional y toda acción atómica condicional que se vuelve elegible se ejecuta eventualmente pues se convierte en "true" con infinita frecuencia.
	- Las políticas de vida dependen de la política de scheduling ya que debe asegurarse de que las acciones sean seleccionadas, asegurando que algo bueno ocurra.

*14- Dado el siguiente programa concurrente, indique cuál es la respuesta correcta (justifique claramente)*  
	int a = 1, b = 0;
	co <await (b = 1) a = 0 > // while (a = 1) { b = 1; b = 0; } oc`
	a) Siempre termina 
	b) Nunca termina 
	**c) Puede terminar o no** --> depende de la política de scheduling, si se utiliza Round Robin podría acabarse el quantum de tiempo se acabe y siempre que se haya evaluado la sentencia await b fuera 0. En cambio si se usa otra política que alterna acciones de procesos, sería fuertemente fair (aunque impráctica).

*15- ¿Qué propiedades que deben garantizarse en la administración de una sección crítica en procesos concurrentes? ¿Cuáles de ellas son propiedades de seguridad y cuáles de vida? En el caso de las propiedades de seguridad, ¿cuál es en cada caso el estado “malo” que se debe evitar?* 
- Las propiedades a garantizarse para el protocolo de entrada y salida (a la sección crítica) son:
	- Exclusión Mutua: a lo sumo un proceso está en su sección crítica. Propiedad de seguridad. Debe evitarse acceder a una sección crítica en simultáneo.
	- Ausencia de Deadlock: si 2 o más procesos tratan de entrar a su sección crítica, al menos uno tendrá éxito. Propiedad de seguridad. Debe evitarse el deadlock.
	- Ausencia de Demora Innecesaria: si un proceso quiere entrar a su sección crítica y el resto está en sus secciones no críticas, entonces el primero no está impedido de entrar a su sección crítica. Propiedad de seguridad. Debe evitarse demora innecesaria, tiempo desperdiciado.
	- Eventual Entrada: un proceso que intenta entrar a su sección crítica tiene posibilidades de hacerlo en algún momento. Propiedad de vida.

*16- Resuelva el problema de acceso a sección crítica para N procesos usando un proceso coordinador. En este caso, cuando un proceso SC[i] quiere entrar a su sección crítica le avisa al coordinador, y espera a que éste le otorgue permiso. Al terminar de ejecutar su sección crítica, el proceso SC[i] le avisa al coordinador. Desarrolle una solución de grano fino usando únicamente variables compartidas (ni semáforos ni monitores).* 
```c
Process worker [id: 1..N]{
	while (true){
		while(TS(ocupado))skip;
		sección crítica;
		aviso = false;
		sección no crítica
	}
}

Process coordinador{
	while (true){
		ocupado = false;
		while(TS(aviso))skip;
	}
}

//utilizo Test and Set (Spin Locks), una solución de grano fino, ya que usar sentencias await son soluciones de grano grueso. No cumple completamente con lo que pide pero es para practicar
```


*17- ¿Qué mejoras introducen los algoritmos Tie-breaker, Ticket o Bakery en relación a las soluciones de tipo spinlocks?* 
- Tie-breaker (funciona bien solo con 2 procesos): ![[Pasted image 20250802194333.png]]
- Ticket (para llevar a N procesos el problema anterior con FA):![[Pasted image 20250802195554.png]]
- Bakery (cuando no existe FA, la solución con SC puede no ser fair y hay que corregirlo):![[Pasted image 20250802200045.png]]
- La mejora es que no genera demora innecesaria.

*18- Analice las soluciones para las barreras de sincronización desde el punto de vista de la complejidad de la programación y de la performance.* 
- Los protocolos busy waiting son complejos y sin clara separación entre variables de sincronización y las usadas para cómputo. Usado para sección crítica y barreras. Es una técnica ineficiente en multiprogramación. Difícil diseñar para probar corrección.

*19- Explique gráficamente cómo funciona una butterfly barrier para 8 procesos usando variables compartidas.*
![[Pasted image 20250804125943.png]]