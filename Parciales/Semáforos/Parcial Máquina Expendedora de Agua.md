Se debe simular el uso de una máquina expendedora de agua con capacidad para 20 botellas y U usuarios. Además, existe un repositor encargado de reponer botellas. Cuando un usuario llega a la máquina, espera su turno (orden de llegada), saca su botella y se retira. Si encuentra la máquina sin botellas le avisa al repositor, espera a que recargue la máquina, toma su botella y se va. Mientras se reponen las botellas se debe permitir que se encolen más usuarios.

```c
Cola cola;
sem espera[U] = ([U]0), mutex =1, terminoReponer = 0, reponer = 0;
int botellas = 20;
bool libre = true;

Process Usuario[id: 0..U]{
	int sig;
	P(mutex);
	if(!libre){
		cola.push(id);
		V(mutex);
		P(espera[id]);
	}
	else{
		libre = false;
		V(mutex);
	}
	if(botellas == 0){
		V(reponer);
		P(terminoReponer);
	}
	botellas--;
	P(mutexCola);
	if(cola.empty()){
		libre = true;
	}
	else{
		sig = cola.pop();
		V(espera[sig]);
	}
	V(mutexCola);
}

Process Repositor{
	while(true){
		P(reponer);
		//repone la máquina
		botellas = 20;
		V(terminoReponer);
	}
}
```