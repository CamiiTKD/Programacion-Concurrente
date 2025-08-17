Simular el uso de un puente con un único carril y sentido por donde puede pasar un vehículo a la vez. Hay N vehículos donde algunos son autos y otros ambulancias. Los autos pasan en orden pero dan prioridad a las ambulancias. Suponga que un auto tarda 5 minutos en pasar.

```c
Monitor AdminPuente{
	cond esperando, esperandoAmbulancia;
	int esperando, esperandoAmbulancia = 0;
	bool libre = true;
	proceso Acceder(prioridad: in bool){
		if(!libre){
			if(prioridad){
				esperandoAmbulancia++;
				wait(esperandoAmbulancia);
			}
			else{
				esperando++;
				wait(esperando);
			}
		}
		else{
			libre = false;
		}
	}
	proceso Salir(){
		if(esperandoAmbulancia > 0){
			esperandoAmbulancia--;
			signal(esperandoAmbulancia);
		}
		else{
			if(esperando > 0){
				esperando--;
				signal(esperando);
			}
		}
		if(esperandoAmbulancia == 0) && (esperando == 0){
			libre = true;
		}
	}
}

Process Vehículo[id: 1..N] {
	bool prioridad;
	prioridad = AutoOAmbulancia(); //asumo funcion, devuelve true si es ambulancia
	AdminPuente.Acceder(prioridad);
	Delay(5 minutos);
	AdminPuente.Salir();
}
```