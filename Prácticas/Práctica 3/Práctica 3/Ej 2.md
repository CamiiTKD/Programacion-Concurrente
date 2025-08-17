```c
monitor BD
cond espera
integer cont = 5;
boolean libre true
integer esperando=0;

Process ingresarBD
If (cont = 0) OR (!libre)
	esperando++
	wait(espera)
	

cont--
if (cont=0)
	libre=false




Process salirBD
if (esperando = 0)    
	 libre=true
cont++
signal(espera)
```

```c
Monitor motor
int usandola = 0
int esperando = 0 
cond espera;

procedure solicitarUso()
	if(usandola = 5) {esperando++;wait(cond);}
	else usandola++
procedure liberarUso()
	if(esperando > 0) {esperando--;signal(cond)}
	else usandola--;
```