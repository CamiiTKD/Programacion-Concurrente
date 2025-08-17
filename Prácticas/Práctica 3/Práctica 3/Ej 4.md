a.
```c
Monitor Corralon{
cond colaCliente;
cond colaEmpleado;
int esperando = 0;
	procedure Llegar(){
		//entregar pedido de compra.
		esperando++;
		signal(colaEmpleado);
		wait(cola);	
	}
	procedure AtenderCliente(){
		if(esperando == 0){
			wait(colaEmpleado);
		}
		esperando--;
		signal(colaCliente);
		//entregar comprobante.
	}
}

Process Empleado{
	for(int i = 1; i < N; i++){
		Corralon.AtenderCliente();
	}
}

Process Cliente[id: 1..N]{
	Corralon.Llegar();
}
```

b.
```c
Monitor Corralon{
cond colaCliente;
cola colaEmpleado;
int empleadosLibres = 0;
int esperando = 0;
	procedure Llegar(idE: out int){
		if(empleadosLibres == 0){
			esperando++;
			wait(cola);
		}
		else{
			empleadosLibres--;
		}
		cola.pop(colaEmpleado, idE);
	}
	procedure Proximo(idE: in int){
		push(colaEmpleado, idE);
		if(esperando > 0){
			esperando--;
			signal(colaCliente);
		}
		else{
			empleadosLibres++;
		}
	}
}

Monitor Escritorio[id: 1..E]{
bool llego = false;
cond Empleado;
cond Cliente;
ListaProducto lista;
Recibo recibo;
	Procedure Atencion(listaProd: in ListaProducto, R: out Recibo){
		lista = listaProd;
		llego = true;
		signal(Empleado); //aviso al empleado que llegué
		wait(); //me duermo hasta que me de el recibo
		R = recibo;
		signal(Empleado); //le aviso al empleado que me llegó el recibo
	}
	
	Procedure EsperarLista(l: out ListaProducto){
		if(!llego){
			wait(Empleado) //me duermo hasta que llegue el cliente
		}
		lista = ListaProd; //recibo los productos del cliente
	}
	
	Procedure EntregarRecibo(r: in Recibo){
		recibo = r;
		signal(Cliente); //le aviso al cliente que agarre el recibo
		wait(Empleado); //me duermo para que el cliente me avise cuando lo recibe
		listo = false; //espero otro cliente
	}
}

Process Empleado[id: 1..E]{
	ListaProducto lista;
	Recibo recibo;
	while(true){
		Corralon.Proximo(id);
		Escritorio.EsperarLista(lista); //espero que me mande los productos
		recibo = armarRecibo(lista);
		Escritorio.EntregarRecibo();
	}
}

Process Cliente[id: 1..N]{
	int idE;
	ListaProducto lista;
	Recibo recibo;
	lista = crafteoLista();
	Corralon.Llegar(idE);
	Escritorio[idE].Atencion(lista, recibo); //entrego la lista y espero el recibo
}
```
c.
```c
Monitor Corralon{
cond colaCliente;
cola colaEmpleado;
int cantidadAtendidos = 0;
int empleadosLibres = 0;
int esperando = 0;
	procedure Llegar(idE: out int){
		if(empleadosLibres == 0){
			esperando++;
			wait(cola);
		}
		else{
			empleadosLibres--;
		}
		cola.pop(colaEmpleado, idE);
	}
	procedure Proximo(idE: in in, seguir: out bool){
		if(cantidadAtendidos == N){
			seguir = false;
		}
		else{
			cantidadAtendidos++;
			seguir = true;
			push(colaEmpleado, idE);
			if(esperando > 0){
				esperando--;
				signal(colaCliente);
			}
			else{
				empleadosLibres++;
			}
		}
	}
}

Monitor Escritorio[id: 1..E]{
bool llego = false;
cond Empleado;
cond Cliente;
ListaProducto lista;
Recibo recibo;
	Procedure Atencion(listaProd: in ListaProducto, R: out Recibo){
		lista = listaProd;
		llego = true;
		signal(Empleado); //aviso al empleado que llegué
		wait(); //me duermo hasta que me de el recibo
		R = recibo;
		signal(Empleado); //le aviso al empleado que me llegó el recibo
	}
	
	Procedure EsperarLista(l: out ListaProducto){
		if(!llego){
			wait(Empleado) //me duermo hasta que llegue el cliente
		}
		lista = ListaProd; //recibo los productos del cliente
	}
	
	Procedure EntregarRecibo(r: in Recibo){
		recibo = r;
		signal(Cliente); //le aviso al cliente que agarre el recibo
		wait(Empleado); //me duermo para que el cliente me avise cuando lo recibe
		listo = false; //espero otro cliente
	}
}

Process Empleado[id: 1..E]{
	ListaProducto lista;
	Recibo recibo;
	bool seguir = true;
	while(seguir){
		Corralon.Proximo(id);
		if(seguir){
			Escritorio.EsperarLista(lista); //espero que me mande los productos
			recibo = armarRecibo(lista);
			Escritorio.EntregarRecibo();
		}
	}
}

Process Cliente[id: 1..N]{
	int idE;
	ListaProducto lista;
	Recibo recibo;
	lista = crafteoLista();
	Corralon.Llegar(idE);
	Escritorio[idE].Atencion(lista, recibo); //entrego la lista y espero el recibo
}
```