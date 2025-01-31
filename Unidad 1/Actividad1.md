# Unidad 1

## Actividad 1


### ¿Qué es un computador digital moderno?
Un computador digital moderno se trata de un dispositivo que cuenta con, minimamente, una CPU (Unidad central de procesamiento) y una forma de almacenamiento. A partir de estos componentes, una computadora es capaz de recibir, procesar y dar resultados a ordenes dadas en binario (lenguaje maquina).

### ¿Cuáles son sus partes?
Como dicho previamente, las partes de una computadora moderna serian, minimamente, una CPU (Unidad central de procesamiento) y alguna forma de memoria (RAM, ROM).


## Actividad 2

### ¿Qué es entonces un programa?
Se refiere a un conjunto de instrucciones, almacenado en la ROM, que se le da a la CPU, quien la traduce y comparte a la memoria, de manera que puedan ser ejecutados.

### ¿Qué es un lenguaje ensamblador?
Se refiere a lenguajes de bajo nivel, es decir, que el programador unicamente puede dar instrucciones sencillas a la maquina, principalmente, operaciones aritmeticas, tomar valores ya inscritos en la memoria, mover valores de un registro a otro, probar booleans, etc. Esto de una forma que sea mas similar a lenguajes aritmeticos o simbolicos utilizados de manera cotidiana.

### ¿Qué es lenguaje de máquina?
Se refiere al lenguaje que entiende la CPU y la Memoria, dada en binarios.


## Actividad 3

### ¿Qué son PC, D y A?
PC es una forma de indicar cuantas lineas de codigo se han ejecutado hasta ahora. A se refiere a Address Register (registro de direccion), y D se refiere a Data Register (registro de datos).

### ¿Para qué los usa la CPU?
La CPU utiliza estas variables para poder ejecutar distintas instrucciones dadas mediante el lenguaje ensamblador. Sirven para poder registrar valores en las n-casillas de RAM disponibles, poder ejecutar operaciones aritmeticas, tomar valores ya inscritos en la memoria, mover valores de un registro a otro, probar booleans, etc.

## Actividad 4

### Considera el siguiente fragmento de código en lenguaje ensamblador:
``` 
@16384
D = A
@16
M = D 
```
### El resultado de este programa es que guarda en la posición 16 de la RAM el valor 16384. Ahora escribe un programa en lenguaje ensamblador que guarde en la posición 32 de la RAM un 100.

#### <ins>Codigo respuesta</ins>
```
@100
D=A
@32
M=D
```
