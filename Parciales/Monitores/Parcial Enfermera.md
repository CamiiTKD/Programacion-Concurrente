Simular la atención de una salita medica para vacunar. Hay una enfermera y 30 pacientes con turno ya asignado (1..30). La enfermera los vacuna de acuerdo a su turno. Cada paciente espera hasta que sea su turno y se dirige al consultorio para ser vacunado, luego se retira. Suponer función Vacunar().

```c
Monitor SalitaMedica{
	cond enfermera, pacientesEsperando[30];
	int turnoActual = -1;
	bool presente[30] = ([30]false);
	procedure Llegue(turno: in int){
		presente[turno] = true;
		if (turno == turnoActual){
			signal(enfermera);
		}
		wait(pacientesEsperando[turno]);
	}
	procedure Proximo(){
		turnoActual++;
		if(!presente[turnoActul]){
			wait(enfermera);
		}
		signal(pacienteEsperando[turnoActual]);
	}
}
Monitor Consultorio{
	cond enfermera, paciente;
	bool llegoPaciente = false;
	procedure SerAtendido(){
		llegoPaciente = true;
		signal(enfermera);
		wait(paciente);
		signal(enfermera);
	}
	procedure EsperarPaciente(){
		if(!llegoPaciente){
			wait(enfermera);
		}
		llegoPaciente = false;
	}
	procedure TerminarAtencion(){
		signal(paciente);
		wait(enfermera);
	}
}

Process Enfermera{
	for[int i=0; i<30; i++]{
		Salita.Proximo();
		Consultorio.EsperarPaciente();
		Vacunar()
		Consultorio.TerminarAtencion();
	}
}

Process Paciente[id: 1..30]{
	SalitaMedica.Llegue();
	Consultorio.SerAtendido();
}
```