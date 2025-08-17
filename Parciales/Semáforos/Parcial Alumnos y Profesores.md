En un examen final hay P alumnos y 3 profesores. Cuando todos los alumnos y profesores llegan comienza el examen. El primer profesor en llegar le da el examen a cada alumno. Cada alumno resuelve el examen, lo entrega y espera a que alguno de los profesores lo corrija y le indique la nota. Los profesores corrigen los exámenes en el orden en que van entregando.

```c
int cant = 0;
int primerP = -1;
sem esperarExamen = 0, primerProfe = 0, mutex = 1, mutexLlegada = 1;
sem seguir[P] = ([P]0), recibirNota[P] = ([P]0), corregir = 0;
Examen vecExamenes[P];
Examen vecResueltos[P];
int vecNotas[P];
Cola cola;
Process Profesor[id: 0..2]{
	int id, nota;
	Examen examen;
	
	P(mutexLlegada);
	cant++;
	if(cant == P + 3){
		V(primerProfe);
	}
	if(primerP == -1){
		primerP = id;
	}
	V(mutexLlegada);
	
	if(primerP == id){
		P(primerProfe);
		for[int i = 0; i < P; i++]{
			vecExamenes[i] = entregarExamen();
			V(seguir[id]);
		}
	}
	
	while(true){
		P(corregir);
		P(mutex);
		id = cola.pop();
		V(mutex);
		examen = vecResueltos[id];
		nota = CorregirExamen(examen);
		vecNotas[id] = nota;
		V(recibirNota[id]);
	}
}
Process Alumno[id: 0..P-1]{
	Examen examen;
	
	P(mutexLlegada);
	cant++;
	if(cant == P + 3){
		V(primerProfe);
	}
	V(mutexLlegada);
	
	P(seguir[id]);
	examen = vecExamenes[id];
	ResolverExamen(examen);
	
	vecResueltos[id] = examen;
	P(mutex);
	cola.push(id);
	V(mutex);
	V(corregir);
	
	P(recibirNota[id]);
	nota = vecNotas[id];
}
```