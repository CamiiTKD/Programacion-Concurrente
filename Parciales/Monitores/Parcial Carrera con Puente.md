Se debe simular una carrera con C corredores donde en la mitad del recorrido hay un puente colgante que puede ser usado por una persona a la vez. Cuando todos los corredores llegan a la línea de llegada, comienza la carrera. Cuando un corredor llega al puente espera su turno por orden de llegada, lo cruza y luego continúa la carrera.

```c
Monitor Carrera{
	cond espera;
	int espera = 0;
	bool libre = true;
	procedure Llegar(){
		espera++;
		if(espera == C){
			espera = 0;
			signal_all(espera);
		}
		else{
			wait(espera);
		}
	}
	procedure AccesoPuente(){
		if(!libre){
			espera++;
			wait(espera);
		}
		else{
			libre = false;
		}
	}
	procedure SalirPuente(){
		if(espera > 0){
			espera--;
			signal(espera);
		}
		else{
			libre = true;
		}
	}
}

Process Corredor[id: 0..C]{
	Carrera.Llegar();
	//le mete pata
	Carrera.AccesoPuente();
	//cruza el puente
	Carrera.SalirPuente();
}
```