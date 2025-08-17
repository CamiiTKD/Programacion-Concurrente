En una sala se juntan 20 conferencistas y un coordinador para una conferencia. Cuando todos llegan a la sala, el coordinador abre la sesión con una presentación de 30 min y luego cada conferencista va dando su presentación de 10 minutos, de a uno y por orden de llegada. Cuando todos terminan, se retiran.

```c
Monitor Sala{
	cond coord, espera, termino;
	int cant = 0;
	int cantExpo = 0;
	procedure Llegar(bool coordinador){
		cant++;
		if(cant == 21){
			signal(coord); //si llegaron todos despierto al coordinador
			if(!coordinador){ //pregunto para no dormir al coordinador de nuevo
				wait(espera);
			}
		}
		else{ //duermo al que venga, corta
			if(coordinador){
				wait(coord);
			}
			else{
				wait(espera);
			}
		}
	}
	procedure terminarPresentación(){
		cantExpo++;
		if(cantExpo == 21){
			signal_all(termino); //si terminaron todos los despierto
		}
		else{
			signal(espera); //llamo al siguiente
			wait(termino); //me duermo hasta que terminen todos
		}
	}
}

Process Conferencista[id: 0..19]{
	Sala.Llegar(false);
	delay(10 min);
	Sala.TerminarPresentación();
	//se va
}
Process Coordinador{
	Sala.Llegar(true);
	delay(30 min);
	Sala.TerminarPresentación();
	//se va
}
```