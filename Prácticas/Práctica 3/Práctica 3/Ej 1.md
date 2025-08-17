a. Si, el código funciona correctamente ya que al querer entrar un auto al puente, se llama al procedimiento entrarPuente() y este se encarga de verificar si hay autos en el puente, si no hay autos en el puente, se incrementa la variable cant y, si hay autos en el puente, se duerme al proceso hasta que el auto que está en el puente disminuya la variable cant y despierte al primer proceso dormido en la cola de espera. Lo único a remarcar es que no se respeta el orden de llegada de los vehículos. Esto se puede solucionar usando Passing the Condition.

b. Se puede simplificar representando el cruce del puente como un procedimiento dentro del monitor. Como el monitor ya está implementado como EM respeta el cruce individual.

```c
Monitor Puente {
    Procedure cruzarPuente() {
        # El auto cruza el puente
    }
}
Process Auto [a:1..M] {
    Puente.cruzarPuente();
}
```

c. La solución del punto B sí respeta el orden de llegada, mientras que la primera no, ya que xavisa que cualquier proceso que está esperando debe competir por el monitor.