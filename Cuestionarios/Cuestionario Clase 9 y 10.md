*1- Explique brevemente los 7 paradigmas de interacción entre procesos en programación distribuida vistos en teoría. En cada caso ejemplifique, indique qué tipo de comunicación por mensajes es más conveniente y qué arquitectura de hardware se ajusta mejor. Justifique sus respuestas.* 
- Master/Worker: Derivado del cliente/servidor, es la implementación del modelo "Bag of Tasks". 
	- Ejemplo: Un renderizador de imágenes distribuido, donde el maestro asigna a cada trabajador una porción de la imagen para renderizar.
	- Comunicación: Pasaje de Mensajes Sincrónico (PMS) o Asincrónico (PMA). La comunicación puede ser sincrónica (el maestro espera a que el trabajador complete la tarea) o asincrónica (el maestro envía tareas a los trabajadores que estén libres). El uso de sockets y colas de mensajes es ideal.
	- Arquitectura: Arquitectura en Estrella (Centralizada). El nodo central (maestro) gestiona toda la comunicación y distribución de tareas, mientras que los nodos periféricos (trabajadores) se conectan a él.
- Heartbeat: Derivado de los pares que interactúan, los procesos se comunican con los vecinos. Todos son workers que trabajan actualizando una parte del resultado, iteran una determinada cantidad de veces (se envían mensajes con datos propios a los vecinos directos y reciben los datos de los vecinos, actualizan la información propia y así continúan con el resto de los vecinos).
	- Ejemplo: Un sistema de tolerancia a fallos en un clúster de servidores, donde cada nodo verifica la salud de sus vecinos.
	- Comunicación: Pasaje de Mensajes Asincrónico (PMA) de bajo nivel. La comunicación es constante y de bajo volumen. La naturaleza asincrónica es crucial para no bloquear la ejecución de los procesos mientras esperan un latido. Se utiliza la mensajería de forma unidireccional.
	- Arquitectura: Anillo Circular o Totalmente Conectada. En un anillo, cada nodo se comunica con su predecesor y sucesor. En una arquitectura totalmente conectada, cada nodo puede monitorear a todos los demás, lo que ofrece mayor redundancia.
	- conociendo el diámetro de la red:![[Pasted image 20250804215107.png]]
	- sin conocer el diámetro de la red:![[Pasted image 20250804215955.png]]
- Pipeline: Derivado del productor/consumidor, procesamiento por etapas (como línea de producción).
	- Ejemplo: Un sistema de procesamiento de datos donde un proceso lee datos de un archivo, otro los limpia y filtra, y un tercero los analiza.
	- Comunicación: Pasaje de Mensajes Asincrónicos (PMA) a través de canales tipo link o pipe. Los canales actúan como colas de mensajes, lo que permite que los procesos se ejecuten a su propio ritmo sin bloquear al productor si el consumidor está ocupado.
	- Arquitectura: Grilla o Anillo Circular. Una grilla de una sola dimensión o un anillo circular son arquitecturas naturales para este patrón, ya que los datos se mueven de forma secuencial de un nodo a otro.
- Probes y Echoes: Permite recorrer estructuras dinámicas (árboles, grafos, etc) recolectando información.
	- Ejemplo: Un algoritmo para encontrar el centro de un árbol de forma distribuida.
	- Comunicación: Pasaje de Mensajes Sincrónico (PMS) o RPC. El patrón de sondeo y eco implica una comunicación de ida y vuelta. El proceso que inicia el sondeo debe esperar el eco, lo que se ajusta a un modelo de comunicación sincrónica. La RPC también es una buena opción, ya que simula llamadas a procedimientos en los nodos.
	- Arquitectura: Árbol o Totalmente Conectada. Este paradigma es natural para arquitecturas en árbol, ya que la propagación y la recolección de información siguen la estructura del árbol.
- Broadcast: Permite alcanzar información global de manera descentralizada. Distribuye trabajo sin coordinador.
	- Ejemplo: La distribución de un nuevo bloque en una red de blockchain a todos los nodos.
	- Comunicación: Pasaje de Mensajes Asincrónico (PMA) con una primitiva de broadcast. La difusión asincrónica es fundamental para no bloquear al proceso emisor. La primitiva de broadcast es más eficiente que enviar mensajes individualmente a cada proceso, especialmente en arquitecturas totalmente conectadas.
	- Arquitectura: Totalmente Conectada o Árbol. En una arquitectura totalmente conectada, el broadcast es trivial. En un árbol, se puede implementar de manera eficiente, donde la raíz envía el mensaje a sus hijos, y estos a los suyos, y así sucesivamente
- Token Passing: Token de control que viaja a través de los procesos. Sirve para recursos compartidos, decisiones distribuidas, etc.
	- Ejemplo: Un algoritmo de exclusión mutua distribuida donde solo el proceso que tiene el token puede entrar en su sección crítica.
	- Comunicación: Pasaje de Mensajes Asincrónicos (PMA) o Sincrónicos (PMS). La comunicación es unidireccional y secuencial. El proceso que termina de usar el token lo envía a su sucesor. La naturaleza asincrónica permite que el proceso receptor se bloquee hasta que reciba el token, mientras que el emisor continúa su ejecución.
	- Arquitectura: Anillo Circular. Esta es la arquitectura más natural para el paso de testigo, ya que el token viaja de forma ordenada y secuencial de un nodo a otro.
- Servidores Replicados: Los servidores manejan recursos compartidos como dispositivos o archivos.
	- Ejemplo: Un clúster de bases de datos donde cada nodo tiene una copia de los datos y las actualizaciones se replican entre ellos.
	- Comunicación: Pasaje de Mensajes Asincrónico (PMA). La comunicación se utiliza para replicar los datos y sincronizar el estado de los servidores. Los clientes pueden usar pasaje de mensajes sincrónico (RPC o Rendezvous) para interactuar con un servidor, y la comunicación interna entre servidores es asincrónica.
	- Arquitectura: Totalmente Conectada. Cada servidor debe ser capaz de comunicarse con todos los demás para replicar los cambios de estado y mantener la consistencia. Esta arquitectura garantiza que cualquier servidor pueda contactar a los demás de manera eficiente.

*2- Describa el paradigma “bag of tasks”. ¿Cuáles son las principales ventajas del mismo?* 
- El concepto bag of tasks supone que un conjunto de workers comparten una bolsa con tareas independientes. Los workers sacan tareas y las ejecutan (hasta pueden generar más).
- Ventajas: provee escalabilidad y facilidad para equilibrar la carga de trabajo.

3- Suponga n2 procesos organizados en forma de grilla cuadrada. Cada proceso puede comunicarse solo con los vecinos izquierdo, derecho, de arriba y de abajo (los procesos de las esquinas tienen solo 2 vecinos, y los otros en los bordes de la grilla tienen 3 vecinos). Cada proceso tiene inicialmente un valor local v. 
a) Escriba un algoritmo heartbeat que calcule el máximo y el mínimo de los n2 valores. Al terminar el programa, cada proceso debe conocer ambos valores. (Nota: no es necesario que el algoritmo esté optimizado). 
b) Analice la solución desde el punto de vista del número de mensajes. 
c) Puede realizar alguna mejora para reducir el número de mensajes? 
d) Modifique la solución de a) para el caso en que los procesos pueden comunicarse también con sus vecinos en las diagonales. 

*4- Explicar la clasifican de las comunicaciones punto a punto en las librerías de Pasaje de Mensajes en general: bloqueante y no bloqueante, con y sin buffering.* 
- ![[Pasted image 20250804222207.png]]
- Send/Receive Bloqueante: para asegurarse de que el valor enviado es el correcto, espera hasta que el dato enviado esté guardado en algún lado seguro antes de devolver el control. Genera ociosidad en el emisor, el cual espera de manera "sincrónica".
	- Sin buffering: la comunicación se asemeja a pasaje de mensaje sincrónico, espera a que el otro proceso reciba el dato enviado. Puede ser más rápido que con buffer si el receptor es rápido, o sea siempre está listo para recibir, ya que ahora la copia extra.
	- Con buffering: reduce el tiempo ocioso con un buffer de por medio, el mensaje se copia en el buffer y el proceso emisor puede continuar trabajando. Necesita memoria extra. Se asemeja a pasaje de mensaje asincrónico.
- Send/Receive No Bloqueante: no espera a que se guarde el dato, puede generar problemas ya que no se asegura la información (un proceso puede modificar el dato antes de que el proceso lo reciba).
	- Sin buffering: más inseguro, ya que hasta que el dato no sea recibido por el otro proceso, el dato no se asegura (no se produce la comunicación).
	- Con buffering: más seguro, se produce una copia del dato. Igualmente hasta que se produce la copia del dato, el mismo va a estar inseguro.

*5- Describa sintéticamente las características de sincronización y comunicación de MPI. Explicar por qué son tan eficientes las comunicaciones colectivas en MPI.* 
- MPI es un estándar que define una librería estándar que puede ser usada desde C o Fortran, define la sintaxis y la semántica de 125 rutinas (todas empiezan con MPI_). Sigue el modelo SPMD (single program multiple data), múltiples procesos corriendo un único programa.
- Comunicación Basada en Mensajes: MPI se basa en un modelo de pasaje de mensajes, donde los procesos se comunican enviando y recibiendo datos a través de canales punto a punto. Hay procesos comunicadores que involucran todos los procesos que se pueden comunicar entre ellos (solo pueden comunicarse dentro de los comunicadores, MPI_COMM_WORLD).
- Sincronización: 
	- Ausencia de memoria compartida: En un sistema de memoria distribuida, no se requieren mecanismos especiales de exclusión mutua, ya que los procesos no comparten variables.
	- Sincronización por bloqueo: Las operaciones de comunicación bloqueante (MPI_Send y MPI_Recv) actúan como puntos de sincronización, ya que los procesos esperan en esas llamadas hasta que el mensaje es enviado o recibido.
- Send bloqueante con buffering (Bsend), Send bloqueante sin buffering (Ssend), Send no bloqueante (Isend). Receive bloqueante (Recv) y Receive no bloqueante (Irecv).
- Las operaciones de comunicación colectiva, MPI las implementa de manera muy eficiente. Permite hacer barreras, broadcast, scatter, gather, reduce, etc. Estas operaciones se deben realizar dentro de un comunicador, todos los procesos del comunicador deben llamar a la rutina colectiva. 
- La ventaja es que posee orden logarítmico, primero 1 proceso se comunica con otro, luego esos 2 con otros 2, luego esos 4 con otros 4 y así...![[Pasted image 20250804234333.png]]
*6- ¿Qué relación encuentra entre el paralelismo recursivo y la estrategia de “dividir y conquistar”? ¿Cómo aplicaría este concepto a un problema de ordenación de un arreglo?* 
- Dividir y conquistar implica paralelismo recursivo, o sea hay un problema recursivo y se divide de manera repetitiva en problemas más pequeños (etapa de dividir) hasta que las tareas puedan encargarse de esos problemas pequeños (conquistar). Luego se van devolviendo los resultados (como con backtracking) y combinando las soluciones. Pueden mapearse subproblemas a procesadores y si no los pueden resolver por su cuenta, crean hijos y los distribuyen.
- En el caso de ordenamiento de un arreglo. Puede utilizarse un mergesort, dividiendo el vector en pedazos, las hojas ordenan los pedazos y los resultados se pasan a los procesos/nodos superiores que a su vez ordenan estos resultados generando nuevos y pasándolos hacia arriba hasta llegar al padre que ordena el total del vector.

*7- a) Cómo puede influir la topología de conexión de los procesadores en el diseño de aplicaciones concurrentes/paralelas/distribuidas? Ejemplifique.* 
*b) Qué relación existe entre la granularidad de la arquitectura y la de las aplicaciones?* 
- La conexión de los procesadores influye directamente sobre los métodos de comunicación y sincronización a usar, para memoria distribuida requerirá pasaje de mensajes, en cambio para memoria compartida probablemente se utilizaran semáforos o monitores. 
- Están las que siguen un esquema UMA, donde las unidades de procesamiento comparten memoria
- Y las que siguen esquema NUMA, donde las unidades de procesamiento poseen memoria propia (más rápido acceso a nivel local, pero más lento el acceso remoto ya que deben pasar por buses). Por ende, hay que ser minuciosos con la alocación de los datos.
- En cuanto a la granularidad, debe evaluarse la arquitectura que se posee, si la comunicación es muy lenta conviene tener muchos procesos que trabajen mucho pero que se comuniquen poco (granularidad gruesa). En cambio si se poseen muchas máquinas con una red de comunicación rápida, conviene tener procesos que hagan poco pero se comuniquen mucho (granularidad fina). 

*8- a) ¿Cuál es el objetivo de la programación paralela?* 
*b) ¿Cuál es el significado de las métricas de speedup y eficiencia? ¿Cuáles son los rangos de valores en cada caso?* 
*c) ¿En qué consiste la “ley de Amdahl”?* 
*d) Suponga que la solución a un problema es paralelizada sobre p procesadores de dos maneras diferentes. En un caso, el speedup (S) está regido por la función S=p-1 y en el otro por la función S=p/2. ¿Cuál de las dos soluciones se comportará más eficientemente al crecer la cantidad de procesadores? Justifique claramente.* 
*e) Suponga que el tiempo de ejecución de un algoritmo secuencial es de 10000 unidades de tiempo, de las cuales sólo el 90% corresponde a código paralelizable. ¿Cuál es el límite en la mejora que puede obtenerse paralelizando el algoritmo? Justifique.* 
- a) Obtener mejores tiempos y resultados ejecutando tareas en simultáneo. Es complejo paralelizar problemas, primero hay que descomponer las tareas y luego mapear los procesos a procesadores, ver qué granularidad va a tener, que tipo de sincronización y comunicación usar, librerías, etc. Busca ganar performance.
- b) 
	- Speedup = Ts/Tp. --> para ver cuántas veces más rápida es la solución paralela respecto a la secuencial (ante mismo tamaño de problema).
		- Tiempo paralelo (Tp): es el tiempo desde que empieza el primer proceso hasta que termina el último (tiempo de ejecución completo).
		- Tiempo secuencial (Ts): es el tiempo que tarda la mejor solución secuencial al problema paralelo. Probado en la mejor máquina (procesador).
		- S óptimo: ![[Pasted image 20250805093119.png]]
		- Rango de valores de Speedup: <0 y Sóptimo> . Speedup lineal, sublineal o superlineal.
	- Eficiencia = S/Sóptimo. --> indica que tan cerca estamos al Speedup óptimo
		- Mide la fracción de tiempo en que los procesadores son útiles.
		- Rango de valores de Eficiencia: <0 y 1>. Si es 1 es speedup perfecto.
- c) Ley de Amdahl: La ley de Amdahl es una fórmula [S = 1/((1−P)+(P/N))] que se utiliza para estimar la máxima mejora teórica en la velocidad de ejecución de una tarea cuando se aumentan los recursos de procesamiento. Se basa en la premisa de que una parte del programa puede ejecutarse en paralelo, mientras que otra parte debe ejecutarse de forma secuencial. Dependiendo del resultado de la fórmula (cuando N tiende a infinito) se puede decir que, incluso con un número infinito de procesadores, la mejora de la velocidad está limitada por la parte secuencial del programa. La ley de Amdahl destaca la importancia de la fracción secuencial de un problema. Un programa con una pequeña porción secuencial puede beneficiarse enormemente del paralelismo.
- d) 
- e) 

9- a) Analizando el código de multiplicación de matrices en paralelo planteado en la teoría, y suponiendo que N=256 y P=8, indique cuántas asignaciones, cuántas sumas y cuántos productos realiza cada proceso. ¿Cuál sería la cantidad para cada operación en la solución secuencial realizada por un único proceso? 
b) Si los procesadores P1 a P7 son iguales, y sus tiempos de asignación son 1, de suma 2 y de producto 3, y si el procesador P8 es 3 veces más lento, ¿cuánto tarda el proceso total concurrente? ¿Cuál es el valor del speedup? ¿Cómo podría modificar el código para mejorar el speedup? 
c) Calcule la Eficiencia que se obtiene en el ejemplo original y en el modificado.
- a) ![[Pasted image 20250805103349.png]]
- b) ![[Pasted image 20250805103935.png]]
- ![[Pasted image 20250805104152.png]]
- 