Simular la atención en un centro de vacunación con 8 puestos para vacunar. Al centro acuden 200 pacientes, cada uno ya conoce el puesto al que debe ir. En cada puesto hay un empleado para vacunar y atiende de acuerdo a la edad del paciente (mayor a menor). Cada paciente espera a que lo llamen, luego se dirige al puesto a ser vacunado y al final se retira. Suponer que existe una función Vacunar() y que cada puesto tiene 25 pacientes asignados.

```c
Monitor SalaEspera{
	cond espera[200], empleado[8];
	cola esperaOrdenada[8]; //vector de 8 colas
	bool empleadoLibre[8] = ([8]true);
	procedure Llegar(id: in int, puesto: in int, edad: in int){
		if(empleadoLibre[puesto]){
			empleadoLibre[puesto] = false;
			signal(empleadoLibre[puesto]);
		}
		else{
			esperaOrdenada.insertarMayorAMenor(id, edad);
			wait(espera[id]);
		}
	}
	procedure Proximo(id: in int){
		if(esperaOrdenada[id].empty()){
			wait(empleado[id]);
		}
		else{
			signal(espera[esperaOrdenada[id]);
		}
	}
}
Monitor Puesto[id: 0..7]{
	cond paciente, empleado;
	bool llegó = false;
	procedure SerAtendido(){
		llegó = true;
		signal(empleado);
		wait(paciente);
		signal(empleado);
	}
	procedure Atender(){
		if(!llegó){
			wait(empleado);
		}
		llegó = false;
		Vacunar(); //lo hago acá porque no encuentro problema
		signal(paciente);
		wait(empleado);
	}
}

Process Paciente[id: 0..199]{
	int puesto = ...;
	int edad = ...;
	SalaEspera.Llegar(id, puesto, edad);
	Puesto[puesto].SerAtendido();
}
Process Empleado[id: 0..7]{
	for[int i=0; i < 25; i++]{
		SalaEspera.Proximo(id);
		Puesto[id].Atender();
	}
}
```


> [!NOTE] ==ACLARACION==
> El profesor (genio de la concurrencia) dice que mejor tener 8 monitores para la sala de espera, así cada uno llega a la sala de espera de cada puesto y maximiza la concurrencia, permitiendo a múltiples empleados llamar a un próximo paciente simultáneamente.
