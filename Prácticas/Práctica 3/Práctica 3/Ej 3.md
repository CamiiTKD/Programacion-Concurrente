a.
```c
Monitor Fotocopiadora {
	procedure Fotocopiar(){
		//Fotocopia;
	}
}

Process Persona[id: 0..N-1]{
	Fotocopiadora.Fotocopiar();
}
```

b. 
```c
Monitor Fotocopiadora {
cond cola;
int esperando = 0;
boolean libre = true;
	procedure Fotocopiar(){
		if(libre){
			libre = false;
			//usar
			if(esperando > 0){
				esperando--;
				signal(cola);
			}
			libre = true;
		}
		else{
			esperando++;
			wait(cola);
		}
	}
}

Process Persona[id: 0..N-1]{
	Fotocopiadora.Fotocopiar();
}
```

c. 

```c
Monitor Fotocopiadora {
cond cola[N];
Cola c;
int esperando = 0;
boolean libre = true;
	procedure Fotocopiar(int id, int edad){
		if(libre){
			libre = false;	
		}	
		else{	
			esperando++;	
			insertarMayorAMenor(c, id, edad);	
			wait(cola[id]);	
		}	
	}
	procedure Liberar(){
		if(esperando > 0){	
			esperando--;	
			signal(cola[c.pop()]);	
		}
		else{
			libre = true;
		}
	}
}

Process Persona[id: 0..N-1]{
	Fotocopiadora.Fotocopiar();
	//usa
	Fotocopiadora.Liberar();
}
```

d. 
```c
Monitor Fotocopiadora {
cond cola[N];
int próximo = 1;
	procedure Solicitar(int id){
		if(id != próximo){
			wait(cola[id]);
		}
	}
	procedure Liberar(){	
		if(próximo != N){	
			próximo++;	
			signal(cola[próximo]);	
		}
	}
}

Process Persona[id: 0..N-1]{
	Fotocopiadora.Solicitar();
	//Usa Fotocopiadora;
	Fotocopiadora.Liberar();
}
```

e.
```c
Monitor Empleado {
cond cola[N];
libre = false;
int esperando = 0;
	procedure Solicitar(){
		if(!libre){
			esperando++;
			wait(cola);
		}
		else{
			libre = false;
		}
	}
	procedure Liberar(){
		if(esperando > 0){
			esperando--;
			signal(cola[próximo]);
		}
		else{
			libre = true;
		}
	}
}

Process Persona[id: 0..N-1]{
	Fotocopiadora.Solicitar();
	//Usa Fotocopiadora;
	Fotocopiadora.Liberar();
}
```

f.
```c
Monitor Empleado {
cola fotocopiadoras;
cond cola;
int cantFotocopiadoras = 10;
int esperando = 0;
	procedure Solicitar(idF: out int){
		if(cantFotocopiadoras == 0){
			esperando++;
			wait(cola);
		}
		else{
			cantFotocopiadoras--;
		}
		pop(fotocopiadoras, idF);
	}
	procedure Liberar(idF: in int){
		push(fotocopiadoras, idF);
		if(esperando > 0){
			esperando--;
			signal(cola);
		}
		else{
			cantFotocopiadoras++;
		}
	}
}

Process Persona[id: 0..N-1]{
	int fotocopiadora;
	Fotocopiadora.Solicitar(fotocopiadora);
	//Usa Fotocopiadora ;
	Fotocopiadora.Liberar(fotocopiadora);
}
```
