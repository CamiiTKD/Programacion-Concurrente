```c
Monitor Aula{
	cond alumnosEsperando, preceptor;
	int cantEsperando = 0;
	bool comenzar = false;
	procedure Llegar(){
		cantEsperando++;
		if(cantEsperando == 45){
			comenzar = true;
			signal(preceptor);
		}
		wait(alumnosEsperando);
	}
	procedure EntregarExamen(examen: out Examen){
		if(!comenzar){
			wait(preceptor);
		}
		cantEsperando--;
		//entrega la hoja
		signal(alumnosEsperando);
	}
}
Monitor Escritorio{
	cola examenes;
	int cantExamenes = 0;
	int N;
	bool seguir = true;
	cond profesora, alumnosEsperando;
	procedure Terminar(examen: in Examen, nota: out int){
		cantExamenes++;
		examenes.push(examen);
		if(seguir){ //solo despierto a la profe si soy el unico por corregir
			signal(profesora);
			seguir = false;
		}
		wait(alumnosEsperando);
		nota = N;
		signal(profesora); //aseguro despertar a la profe solo cuando me da la nota
	}
	procedure agarrarParcial(examen: out Examen){
		if(cantExamenes == 0){
			wait(profesora);
		}
		examen = examenes.pop();
		cantExamenes--;
	}
	procedure darNota(nota: in int){
		N = nota;
		signal(alumnosEsperando);
		wait(profesora);
		if(cantExamentes == 0){
			seguir = true;
		}
	}
}

Process Profesora{
	int nota;
	Examen examen;
	for[int i = 0; i < 45; i++]{
		Escritorio.agarrarParcial(examen);
		nota = corregirExamen(examen);
		Escritorio.darNota(nota);
	}
}
Process Alumno[id: 1..45]{
	int nota;
	Examen examen;
	Aula.Llegar(examen);
	//llorar al ver el enunciado
	Escritorio.Entregar(examen, nota);
}
Process Preceptor[]{
	for[int i = 0; i < 45; i++]{
		//lamerse un dedo de la forma más desagradable posible
		//agarrar el parcial
		Aula.EntregarExamen();
	}
}
```
