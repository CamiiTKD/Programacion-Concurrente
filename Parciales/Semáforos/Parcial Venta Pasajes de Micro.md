Una empresa de turismo posee un micro con capacidad para 50 personas. Hay un único vendedor que atiende a los C clientes (C > 50) que intentan comprar un pasaje por orden de llegada, si aún hay lugares se le indica el número de asiento que le tocó. Cada cliente, luego de ser atendido por el vendedor, se dirige al micro para subir en caso de que le hayan dado un asiento, sino se va sin viajar. El micro espera a los 50 pasajeros para arrancar.

```c
Cola cola;
sem pasajeros[C] = ([C]0), mutexCola = 1, hayCliente = 0, atendidos[C] = ([C]0);
sem abordar = 0;
Process Vendedor{
	int idC;
	int cant = 0;
	for[int i = 0; i < C; i++]{
		P(hayCliente);
		P(mutexCola);
		idC = cola.pop();
		V(mutexCola);
		if(cant < 50){
			cant++;
			pasajeros[idC] = darAsiento();
		}
		V(atendidos[idC]);
	}
}

Process Cliente[id: 0..C]{
	int asiento;
	P(mutexCola);
	cola.push(id);
	V(mutexCola);
	V(hayCliente);
	P(atendidos[id]);
	if(pasajeros[id] > 0){
		V(abordar);
	}
}

Process Micro{
	for[int i = 0; i < 50; i++]{
		P(abordar);
	}
	//parte
}
```