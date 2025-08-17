```c
Monitor Equipo[id: 0..3]{
	cond JugadoresEsperando;
	int cantJugadoresEsperando = 0;
	Procedure Llegar(cancha: out int){
		cantJugadoresEsperando++;
		if(cantJugadoresEsperando = 5){
			cancha = Coordinador.EntregarCancha();
			signal_all(JugadoresEsperando);
		}
		else{
			wait(JugadoresEsperando);
		}
	}
}
Monitor Coordinador{
int cancha = 1;
int cantEquipos = 0;
	Procedure EntregarCancha(cancha: out int){
		cantEquipos++;
		if(cantEquipos < 3){
			cancha = 1;
		}
		else{
			cancha = 2;
		}
	}
}

Monitor Cancha[id: 1..2]]{
	cond Esperando;
	int cantEsperando = 0;
	Procedure Llegar(){
		cantEsperando++;
		if(cantEsperando = 10){
			signal_all(esperando);
		}
		else{
			wait(Esperando);
		}
	}
}

Process Jugador[id: 1..20]{
	int equipo = DarEquipo();
	int cancha;
	Equipo[equipo].Llegar(cancha);
	Cancha[cancha].Llegar();
	delay(90 minutitos muchachos);
	//se van todos chivados
}
```