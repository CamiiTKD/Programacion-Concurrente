*1- Defina y diferencie programa concurrente, programa distribuido y programa paralelo.* 
- Programa concurrente: es un programa que utiliza sincronización y comunicación para trabajar en múltiples tareas en simultáneo.
- Programa distribuido: es un programa concurrente comunicado por mensajes. suele trabajarse sobre arquitecturas tipo cluster(distintos procesadores conectados en una red de interconexión).
- Programa paralelo: es un programa concurrente que trabaja apoyado por el hardware(múltiples cores) ejecutando múltiples tareas en paralelo.

*2- Marque al menos 2 similitudes y 2 diferencias entre los pasajes de mensajes sincrónicos y asincrónicos.*
- Similitudes: Usan canales unidireccionales, funcionan para envío de mensajes en memoria distribuida.
- Diferencias: PMS usa canales de tipo link y el envío de mensajes es sincrónico, se bloquea hasta que el receptor reciba el mensaje. PMA usa canales de tipo mailbox y el envío de mensajes es asincrónico, envía y continúa.

*3- Analice qué tipo de mecanismos de pasaje de mensajes son más adecuados para resolver problemas de tipo Cliente/Servidor, Pares que interactúan, Filtros, y Productores y Consumidores. Justifique claramente su respuesta.* 
- Cliente/Servidor:
	- Para problemas de tipo Cliente/Servidor, tanto el Pasaje de Mensajes Asincrónico (PMA) como el Sincrónico (PMS) pueden ser adecuados, pero existen ventajas y desventajas en cada caso:
		- PMA: Permite mayor concurrencia, ya que el cliente puede seguir trabajando después de enviar una solicitud al servidor sin esperar la respuesta. Esto es útil en situaciones donde el cliente no necesita la respuesta inmediatamente.
		- PMS: Es más simple de implementar que el PMA, ya que no requiere un mecanismo para almacenar mensajes en búfer. Sin embargo, puede limitar la concurrencia.
	- RPC (Llamado a Procedimiento Remoto) y Rendezvous son alternativas más adecuadas para problemas Cliente/Servidor que los mecanismos básicos de paso de mensajes. Estos mecanismos combinan la interfaz procedural de los monitores con el paso de mensajes implícitos, simplificando la programación y ocultando la complejidad de la comunicación entre procesos.
- Pares que interactúan:
	- Para problemas de tipo Pares que interactúan, donde los procesos necesitan intercambiar información y sincronizarse entre sí, el PMS puede ser más adecuado:
		- PMS: Permite una sincronización más directa entre los pares, ya que la operación de envío bloquea al emisor hasta que el receptor recibe el mensaje. Esto simplifica la lógica de sincronización en comparación con el PMA.
		- PMA: Aunque se puede utilizar, requiere mecanismos adicionales para garantizar la sincronización y evitar condiciones de carrera.
	- La comunicación guardada, presente en lenguajes como CSP, ofrece una solución eficiente para la interacción entre pares. Esta técnica permite a los procesos esperar selectivamente mensajes de diferentes fuentes, lo que facilita la programación de interacciones complejas.
- Filtros:
	- Para problemas de tipo Filtros, donde los procesos se conectan en una cadena para procesar datos secuencialmente, el PMA es generalmente más adecuado:
		- PMA: Permite mayor concurrencia, ya que cada filtro puede trabajar en su parte del flujo de datos sin esperar a que los demás terminen. Esto se debe a que los canales actúan como colas de mensajes, permitiendo que los filtros envíen datos sin bloquearse.
		- PMS: Puede generar bloqueos y limitar la concurrencia, ya que cada filtro debe esperar a que el siguiente esté listo para recibir antes de poder enviar.
- Productores Consumidores:
	- Para problemas de tipo Productores y Consumidores, donde un proceso genera datos que son consumidos por otro, tanto PMA como PMS pueden ser adecuados, dependiendo de la necesidad de concurrencia:
		- PMA: Ofrece mayor concurrencia, ya que el productor puede seguir generando datos sin esperar a que el consumidor los procese. Esto se logra utilizando un canal como buffer entre el productor y el consumidor.
		- PMS: Si bien reduce la concurrencia, puede ser útil para controlar el flujo de datos entre el productor y el consumidor, asegurando que el productor no genere datos más rápido de lo que el consumidor puede procesar.

*4- Indique por qué puede considerarse que existe una dualidad entre los mecanismos de monitores y pasaje de mensajes. Ejemplifique* 
- Se puede considerar que existe una dualidad entre los mecanismos de monitores y pasaje de mensajes porque cada uno puede simular al otro. Esto significa que cualquier problema de sincronización que se pueda resolver con monitores también se puede resolver con pasaje de mensajes, y viceversa. Un monitor puede ser visto como un proceso que recibe mensajes y envía respuestas. Del mismo modo, el pasaje de mensajes puede ser simulado con monitores que gestionan un buffer para los mensajes. La preferencia en el uso de monitor o de pasaje de mensajes depende de la arquitectura(memoria compartida para el primero y memoria distribuida para el segundo, ya que se vuelven más eficientes). 
- Ej: problema cliente/servidor: en este caso sería necesario un canal para requerimientos, que permita enviar mensajes indicando id, pedido y parámetros. Además, se requerirán n canales privados para que el servidor le responda a los clientes que realizan pedidos. El servidor lee el requerimiento, lo procesa y envía la respuesta por el canal del id que recibió en el mensaje.
- Ej: problema productor/consumidor. 
- Monitores: podríamos tener un monitor que funcione como buffer, con procedimientos insertar() y extraer(). 
	- El productor llama a BufferMonitor.insertar(item). Si el búfer está lleno, se bloquea en la condición no_lleno. Cuando el consumidor extrae un elemento, señaliza (signal()) no_lleno, despertando al productor.
	- El consumidor llama a BufferMonitor.extraer(). Si el búfer está vacío, se bloquea en la condición no_vacio. Cuando el productor inserta un elemento, señaliza (signal()) no_vacio, despertando al consumidor.
- Pasaje de mensajes: se introduce un tercer proceso, el buffer-monitor(ya que no hay monitores). Este proceso tiene su propio espacio de memoria y se encarga de recibir mensajes de los productores y de enviar mensajes a los consumidores.
	- El productor envía un mensaje producir(item) al buffer-monitor. Si el búfer del monitor está lleno, el monitor simplemente ignora la petición o la pone en una cola de espera.
	- El consumidor envía un mensaje consumir() al buffer-monitor. Si el búfer del monitor está vacío, el monitor no le envía nada y el consumidor se bloquea hasta que reciba un mensaje. Cuando hay un item, el monitor envía un mensaje respuesta(item) al consumidor.

*5- ¿En qué consiste la comunicación guardada (introducida por CSP) y cuál es su utilidad? Describa cómo es la ejecución de sentencias de alternativa e iteración que contienen comunicaciones guardadas.*
- CSP (Communicating Sequential Processes): la idea es la de utilizar pasaje de mensaje sincrónico agregando comunicación guardada, pasaje de mensaje con espera selectiva. Usan canales de tipo link unidireccionales. Sentencias de entrada(? o query) y de salida(! o shriek o bang).
- Las operaciones de comunicación(? y !) pueden ser guardadas, es decir hacer AWAIT hasta que una condición sea verdadera. El do e if de CSP usan los comandos guardados de Dijkstra(B-->S), o sea una condición booleana y un conjunto de sentencias asociadas. Trabaja de manera no determinística.
- Funciona para envío y recepción de mensajes (en la práctica sólo para recepción)
- B; C-->S; --> "Guarda"
- B: condición booleana.
- C: sentencia de comunicación.
- Es exitosa si B es true y si C puede ejecutar inmediatamente.
- Falla si B es false.
- Se bloquea si B es true pero C no se puede ejecutar (la comunicación).
- IF: se evalúan las guardas, se elige una exitosa de manera no determinística. Cuando todas las guardas fallan, se sale del if. Si hay algunas en estado de bloqueo (y no hay ninguna exitosa) se queda esperando a que pasen ser exitosas o fallen.![[Pasted image 20250804125252.png]]
- DO: igual que el if pero sigue loopeando, solo sale del loop si ninguna cumple(todas fallan).

*6- Marque similitudes y diferencias entre los mecanismos RPC y Rendezvous. Ejemplifique para la resolución de un problema a su elección.* 
- Similitudes: ambos trabajan con canales bidireccionales, de tipo link y sincrónicos. Ideales para programar aplicaciones cliente/servidor. Poseen una interfaz tipo monitor con operaciones exportadas a través de llamadas externas con mensajes sincrónicos.
- Diferencias: difieren en la manera de servir la invocación a operaciones, RPC tiene módulos que exportan procedimientos y procesos que los llaman. Cada vez que se va a atender un llamado a un procedimiento, se genera un nuevo proceso que resuelve la invocación al procedimiento, devuelve los resultados y luego muere (Ej: JAVA con RMI). Rendezvous tiene procesos que trabajan continuamente e invocan operaciones que resuelven otros procesos. Estos, en algún momento resuelven el pedido y devuelven por parámetro los resultados (Ej: ADA).
- RPC solo brinda comunicación, debe implementarse la sincronización ya que comparten memoria dentro de un mismo módulo.
- Ej: RPC (TimeServer)![[Pasted image 20250804155522.png]]
- Rendezvous combina comunicación y sincronización. Los módulos son procesos. La comunicación guardada en Rendezvous agrega a sus guardas(comunicación guardada) expresiones de scheduling.![[Pasted image 20250804164908.png]]
- Ej: Filósofos Comensales ![[Pasted image 20250804170008.png]]

*7- Considere el problema de lectores/escritores. Desarrolle un proceso servidor para implementar el acceso a la base de datos, y muestre las interfaces de los lectores y escritores con el servidor. Los procesos deben interactuar:* 
*a) con mensajes asincrónicos;* 
*b) con mensajes sincrónicos;* 
*c) con RPC;* 
*d) con Rendezvous.* 
- Es demasiado.

*8- Modifique la solución con mensajes sincrónicos de la Criba de Eratóstenes para encontrar los números primos detallada en teoría de modo que los procesos no terminen en deadlock.* 
- La Criba recorre los N números y va descartando los que no son primos, primero los múltiplos de 2, luego los de 3 y así. El deadlock de la solución se da cuando los procesos (menos Criba 1) se quedan esperando números, se puede modificar con centinelas (o sea el predecesor envía 0 cuando ya no hay más números).
- ![[Pasted image 20250804132914.png]]
```c
Process Criba[1]
{
    int i;
    for [i = 3 to n by 2] Criba[2] ! (i); // Envía los números que no son múltiplos de 2 al primer filtro
    Criba[2] ! (0); //manda 0 si no hay más números
}

Process Criba[i: 2 to L]
{
    int p, proximo;
    Criba[i-1] ? (p);
    if (p == 0) { //recibe 0 si no hay más números
        if (i < L) Criba[i+1] ! (0); //si es el último proceso propaga el 0
    }
    else {
	    do
	        Criba[i-1] ? (proximo) -->
		        if (proximo == 0) && (i < L) Criba[i+1] ! (0); //si es 0 y hay más procesos, lo propaga 
				else 
					if((proximo MOD p) != 0) && (i < L) --> Criba[i+1] ! (proximo); //si cumple, lo pasa
	    od
	}
}
```

*9- Suponga que N procesos poseen inicialmente cada uno un valor. Se debe calcular la suma de todos los valores y al finalizar la computación todos deben conocer dicha suma.* 
*a) Analice (desde el punto de vista del número de mensajes y la performance global) las soluciones posibles con memoria distribuida para arquitecturas en estrella (centralizada), anillo circular, totalmente conectada, árbol y grilla bidimensional.* 
*b) Escriba las soluciones para las arquitecturas mencionadas.* 
- a) 
	- Centralizada: los mensajes al coordinador se envían casi al mismo tiempo(en paralelo), solo el receive del coordinador demora mucho ya que también es secuencial. Nro de mensajes lineal ((2xN)-1).
	- Anillo: solución lenta, cada proceso debe esperar recibir el mensaje del anterior secuencializando el problema. Nro de mensajes lineal (2x(N-1)).
	- Simétrica: es la más corta y sencilla de programar, pero usa el mayor número de mensajes, el overhead de la comunicación acota el speedup (Nx(N-1)).
	- Árbol: los procesos hoja envían sus valores a sus padres, quienes los suman y los envían a su vez a sus propios padres, y así sucesivamente hasta la raíz. El tiempo de ejecución es proporcional a la altura del árbol, lo que lo hace muy eficiente. Nro de mensajes (2x(N-1)).
	- Grilla Bidimensional: La suma se calcula en dos etapas. Primero, se realiza la suma de los elementos de cada fila. Luego, los resultados de cada fila se suman en una columna. Finalmente, el resultado total se propaga a todos los procesos. El tiempo de ejecución está dominado por la dimensión de la grilla. Nro de mensajes (Nx(N−1))
- b)
```c
**Centralizada:**

Process Coordinador{
	chan valores(int), resultados[n-1] (int);
	int suma = mi_valor; int nuevo;
	for (i=1 to n-1) {
		receive valores (nuevo);
		suma = suma + nuevo;
	}
	// Propagación del resultado
	for (i=1 to n-1) {
		send resultados [i] (suma);
	}
}

Process [id: 1 .. n-1] {
    int v; int suma;
    send valores(v);
    receive resultados[id](suma);
}
-----------------------------------------------------------------------------------------------------------
**Anillo**

Process proceso{
	chan valores[n](int); int suma = mi_valor;
    send valores[1](suma);
    receive valores[0](suma);
    send valores[1](suma);
}

Process proceso[id: 1 .. n-1]{
    int v, suma;
    receive valores[id](suma);
    suma = suma + v;
    send valores[(id+1) mod n](suma);
    receive valores[id](suma);
    if (id < n-1) send valores[id+1](suma);
}
-----------------------------------------------------------------------------------------------------------
**Simétrica**

Process proceso[id 0 .. n-1]{
	chan valores[n](int);
    int v, nuevo;
    int suma = v;
    for (k=0 to n-1 st k!=id) //envío a todos menos a mi mismo
        send valores[k](v);
    for (k=0 to n-1 st k!=id)
    {
        receive valores[id](nuevo);
        suma = suma + nuevo;
    }
}
-----------------------------------------------------------------------------------------------------------
**Árbol**
//hecho por Gemini con muchas funciones mágicas.
Process Arbol_Proceso[id: 0 .. n-1] {
    // Fase de Reducción (suma hacia la raíz)
    if (soy_hoja()) {
        send(mi_padre, mi_valor);
    } else {
        receive(mi_hijo_izquierdo, valor_izq);
        receive(mi_hijo_derecho, valor_der);
        suma_parcial = mi_valor + valor_izq + valor_der;
        if (no_soy_raiz()) {
            send(mi_padre, suma_parcial);
        } else {
            suma_final = suma_parcial;
        }
    }
    // Fase de Difusión (suma desde la raíz)
    if (soy_raiz()) {
        send(mi_hijo_izquierdo, suma_final);
        send(mi_hijo_derecho, suma_final);
    } else {
        receive(mi_padre, suma_final);
        if (no_soy_hoja()) {
	        if(tengo_hijo_izquierdo()){
	            send(mi_hijo_izquierdo, suma_final);
	        }
	        if(tengo_hijo_derecho()){
	            send(mi_hijo_derecho, suma_final);
            }
        }
    }
}
-----------------------------------------------------------------------------------------------------------
**Grilla**
//hecho por Gemini, no lo voy a revisar.
Process Grilla_Proceso(i, j) {
    // Fase 1: Suma por filas (asumiendo un anillo por cada fila)
    if (j == 0) {
        suma_fila = mi_valor;
        send((i, j+1), suma_fila);
    }
    receive((i, j-1), valor_recibido);
    if (j != 0) {
        suma_fila = valor_recibido + mi_valor;
        send((i, j+1), suma_fila);
    } else {
        suma_fila_final = valor_recibido;
    }
    // Fase 2: Suma por la primera columna
    if (j == 0) {
        if (i == 0) {
            suma_total = suma_fila_final;
            send((i+1, j), suma_total);
        }
        receive((i-1, j), valor_recibido);
        if (i != 0) {
            suma_total_parcial = valor_recibido + suma_fila_final;
            send((i+1, j), suma_total_parcial);
        } else {
            suma_final = valor_recibido;
        }
    }
    // Fase 3: Propagación de la suma total
    if (j == 0 && i == 0) {
        send((i+1, j), suma_final);
        send((i, j+1), suma_final);
    }
    // Lógica para que cada proceso la propague a sus vecinos
    // ...
}
```

*10- Describa sintéticamente las características de sincronización y comunicación de ADA.*
- Ada posee tareas con primitivas para sincronización, una tarea puede aceptar(accept) una comunicación con otro proceso. Las operaciones exportadas se llaman entry, se llaman haciendo entry call. Rendezvous es el mecanismo principal de comunicación a pesar de que tiene otros no utilizados en la materia [Tarea.entry(parámetros)]. Combina la interfaz procedural de los monitores con el paso de mensajes. La comunicación es sincrónica. El proceso emisor (el que realiza la llamada) se bloquea en la entrada del proceso receptor hasta que este último acepta la llamada y la procesa.
- Exclusión Mutua: La exclusión mutua se maneja de forma inherente dentro del Rendezvous. Cuando un proceso acepta una llamada en un bloque accept, la ejecución del cuerpo de este bloque es atómica y exclusiva, impidiendo que otros procesos accedan a los datos internos de la tarea.
- Ada ofrece una sentencia select que permite a una tarea esperar de forma no determinista a múltiples eventos, como llamadas a diferentes entradas o la expiración de un temporizador.
- En resumen, la sincronización es el "cómo" y el "cuándo" de la interacción (bloqueo, espera, exclusión mutua), mientras que la comunicación es el "qué" se intercambia (los datos a través de los parámetros). Ambos ocurren dentro del mismo evento de Rendezvous.