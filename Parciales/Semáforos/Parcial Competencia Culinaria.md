En una competencia culinaria se juntan 1 jurado y C concursantes. Una vez que llegan todos los concursantes, el jurado les asigna una receta. Los concursantes cocinan el plato y lo exhiben al jurado a medida que van terminando. El jurado asigna un puntaje a cada concursante, el cual lo guarda para su CV.

```c
sem llegada = 0, mutex = 1, mutexTermino = 1, termino = 0;
Receta vecRecetas[C];
Plato vecPlatos[C];
int Notas[C];
sem seguir[C] = 0;
Cola cola, termino;
Process Jurado{
	int id;
	Plato plato;
	int nota;
	
	for[int i = 0; i < C; i++]{
		P(llegada);
	}
	for[int i = 0; i < C; i++]{
		P(mutex);
		id = cola.pop();
		V(mutex);
		vecRecetas[id] = otorgarReceta();
		V(seguir[id]);
	}
	for[int i = 0; i < C; i++]{
		P(termino);
		P(mutexTermino);
		id = termino.pop();
		V(mutexTermino);
		plato = vecPlato[id];
		nota = analizarPlato(plato);
		Notas[id] = nota;
		V(seguir[id]);
	}
}
Process Concursante[id: 0..C]{
	Receta receta;
	Plato plato;
	int puntaje;
	
	P(mutex);
	cola.push(id);
	V(mutex);
	V(llegada);
	
	P(seguir[id]);
	receta = vecRecetas[id];
	plato = Cocinar();
	
	vecPlatos[id] = plato;
	P(mutexTermino);
	termino.Push(id);
	V(mutexTermino);
	V(termino);
	
	P(seguir[id]);
	nota = notas[id];
}
```