Simular la atención de una planta verificadora de vehículos, donde se revisan cuestiones de seguridad de a uno por vez. Hay N vehículos a verificar, algunos autos y otros ambulancias. Antes de ser verificados, los autos deben hacer el pago correspondiente en la caja de la planta, donde le entregarán el recibo. Las ambulancias no pagan. Los vehículos son atendidos de acuerdo al orden de llegada pero dando prioridad a las ambulancias.

```c
Cola colaCaja, colaPrioridad, colaAuto;
sem mutex = 1, mutexPrioridad = 1, mutexAuto = 1, recibo[N] = ([N]0);
sem Caja = 0, verificar = 0, termino[N] = ([N]0);
Recibo vecRecibo[N];

Process Caja{
	int id;
	Recibo recibo
	while(true){
		P(Caja);
		P(mutex);
		id = cola.pop();
		V(mutex);
		recibo = Cobrar(id);
		vecRecibo[id] = recibo;
		V(recibo[id]);
		P(mutexAuto);
		colaAuto.push(id); //esto podría hacerlo el vehículo una vez pagó
		V(mutexAuto); //yo creo que lo anuncia la caja,no tiene que hacerlo el auto
		V(verificar); 
	}
}
Process Verificador{
	int id;
	for[int i = 0; i < N; i++]{
		P(verificar);
		if(colaPrioridad.empty()){
			P(mutexAuto);
			id = colaAuto.pop();
			V(mutexAuto);
		}
		else{
			P(mutexPrioridad);
			id = colaPrioridad.pop();
			V(mutexPrioridad);
		}
		Verificar(id);
		V(termino[id]);
	}
}
Process Vehículo[id: 0..N-1]{
	bool ambulancia = ...;
	Recibo recibo;
	if(!ambulancia){
		P(mutex);
		cola.push(id);
		V(mutex);
		V(Caja);
		P(recibo[id]);
		recibo = vecRecibo[id];
	}
	else{
		P(mutexPrioridad);
		colaPrioridad.push(id);
		V(mutexPrioridad);
		V(verificar);
	}
	P(termino[id]);
}
```