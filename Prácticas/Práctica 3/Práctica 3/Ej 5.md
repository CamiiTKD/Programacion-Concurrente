```c
Monitor Tarea{
int cantAlumnos = 0, nota = 26;
cond Profesor, colaAlumnos[50], colaTerminados[25];
cola Alumnos, GruposTerminados;
int vectorNroPorAlumno[50], vectorGruposTerminando[25] = 0[25];
	Procedure Llegar(id: in int){
		cantAlumnos++;
		Alumnos.push(id);
		if(cantAlumnos == 50){
			signal(Profesor);
		}
	}
	Procedure EsperarAlumnos(){
		if(cantAlumnos < 50){
			wait(Profesor);
		}
	}
	Procedure RecibirNroGrupo(id: in int, nro: out nro){
		wait(colaAlumnos[id]);
		nro = vectorNroAlumnos[id];
	}
	Procedure AsignarGrupos(){
		int id, nro;
		for[int i = 0; i < 25; i++]{
			nro = AsignarNroGrupo();
			for[int j = 0; i < 2; i++]{
				id = Alumnos.pop();
				vectorNroPorAlumno[id] = nro;
				signal(vectorNroPorAlumno[id]);
			}
		}
	}
	Procedure Terminar(id: in int, nro: in int, notaGrupo: out int){
		vectorGruposTerminados[nro]++;
		if(vectorGruposTerminados[nro] < 2){
			wait(colaTerminados[nro]);
		}
		else{
			nota--;
			signal(colaTerminados[nro]);
		}
		notaGrupo = nota;
	}
}

Process Alumno[id: 1..50]{
	int nota, nro;
	Tarea.Llegar();
	Tarea.RecibirNroGrupo(nro):
	//a laburar
	Tarea.Terminar(id,nro,nota);
}

Process Profesor{
	Tarea.EsperarAlumnos();
	Tarea.AsignarGrupos();
}
```
