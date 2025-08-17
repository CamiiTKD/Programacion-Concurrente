```c
Monitor Carrera{
	cond espera;
	cond esperandoAwa;
	int esperando;
	procedure Llegar(){
		esperando++;
		if(esperando < C){
			wait(espera);
		}
		else{
			signal_all(espera);
		}
	}
	procedure LlegarMaquina(){
		if(!maquinaDisponible){
			esperando++;
			wait(esperandoAwa);
		}
		else{
			maquinaDisponible = false;
		}
	}
	procedure LiberarMaquina(){
		if(esperando > 0){
			esperando--;
			signal(esperandoAwa);
		}
		else{
			maquinaDisponible = true;
		}
	}
}
Monitor Maquina{
	int cantAwa = 20;
	cond reponedor, espera;
	procedure Reponer(){
		if(cantAwa > 0){
			wait(reponedor);
		}
		cantAwa = 20;
		signal(espera);
	}
	procedure DameAwa(){
		if(cantAwa == 0){
			signal(reponedor);
			wait(espera);
		}
		cantAwa--;
	}
}

Process Corredor[id: 0..C]{
	Carrera.Llegar();
	//corren re picados
	Carrera.LlegarMaquina();
	Maquina.DameAwa();
	Carrera.LiberarMaquina();
}

Process Reponedor{
	while(true){
		Maquina.Reponer();
	}
}
```