En una clínica hay un médico que debe atender a 20 pacientes de acuerdo al turno de cada uno (debe atender primero al turno i para atender al i+1). Cada paciente ya conoce su turno, al llegar espera hasta que el médico lo llame para ser atendido, se dirige al consultorio y luego espera hasta que el médico lo termine de atender para retirarse.

```c
sem llegada = 0, terminarAtencion = 0;
sem esperaTurno[20]= ([20]0), liberoConsultorio = 0;
int vecTurno[20];

Process Médico{
	int id;
	for[int i = 0; i < 20; i++]{
		V(esperaTurno[i]); //aviso que pase
		P(llegada); //espero a que entre
		id = vecTurno[i];
		Atender(id); //abrí la boca *le mete el palito de helado hasta la traquea*
		V(terminarAtencion);
		P(liberoConsultorio); //espera a que se vaya el paciente.
	}
}
Process Paciente[id: 0..19]{
	int turno = ...;
	vecTurno[turno] = id;
	P(esperaTurno[turno]); //espero que me llamen
	V(llegada); //aviso que paso
	P(terminarAtención);
	V(liberoConsultorio); //aviso que me voy
}
```