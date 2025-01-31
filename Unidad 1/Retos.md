## Retos

Este reto consiste en la elaboración de una serie de programas en lenguaje ensamblador.

- Para realizar las pruebas puedes utilizar uno de los simuladores que puedes encontrar [aquí](https://www.nand2tetris.org/software).
- El simulador que hemos venido utilizando es [este](https://nand2tetris.github.io/web-ide/chip/).
- Utiliza Google Chrome o Microsoft Edge para abrir el simulador.
- Cada que modifiques el programa, asegúrate de guardar el archivo, actualizar la página web y cargar el archivo nuevamente.

### 1) Carga en D el valor 1978.

```
@1978
D=A
```

### 2) Guarda en la posición 100 de la RAM el número 69.

```
@69
D=A
@100
M=D
```

### 3) Guarda en la posición 200 de la RAM el contenido de la posición 24 de la RAM

**Comentario de codigo**:  Asignar un valor aleatorio a 24 previo a correr el programa.

```
@24
D=M
@200
M=D
```

### 4) Lee lo que hay en la posición 100 de la RAM, resta 15 y guarda el resultado en la posición 100 de la RAM.

```
@100
D=M
@15
D=D-A
@100
M=D
```

### 5) Suma el contenido de la posición 0 de la RAM, el contenido de la posición 1 de la RAM y con la constante 69. Guarda el resultado en la posición 2 de la RAM.

```
@0
D=M
@1
D=D+M
@69
D=D+A
@2
M=D
```

### 6) Si el valor almacenado en D es igual a 0 salta a la posición 100 de la ROM.

```
@100
D;JEQ
```

### 7) Si el valor almacenado en la posición 100 de la RAM es menor a 100 salta a la posición 20 de la ROM.

```
@100
D=M
D=D-A
@20
D;JLT
```

### 8) Considera el siguiente programa:

```
@var1
D = M
@var2
D = D + M
@var3
M = D
```

- ¿Qué hace este programa?  
R/ Este programa suma los valores de var1 y var2, y tras esto pone el resultado en @var3

- En qué posición de la memoria está var1, var2 y var3? ¿Por qué en esas posiciones?  
R/ Las posiciones no pueden ser determinadas segun el problema, debido a que en ningun momento esto es particularmente relevante aritmeticamente. Sabiendo esto, y en terminos de logica, se podria suponer que sus posiciones son 0, 1 y 2, respectivamente.

### 9) Considera el siguiente programa:

```
i = 1
sum = 0
sum = sum + i
i = i + 1
```

La traducción a lenguaje ensamblador del programa anterior es:

```
// i = 1
@i
M=1
// sum = 0
@sum
M=0
// sum = sum + i
@i
D=M
@sum
M=D+M
// i = i + 1
@i
D=M+1
@i
M=D
```

- ¿Qué hace este programa?  
R/ Este programa hace que en la posicion 0 de la RAM haya un 1, y que en la posicion 1 de la RAM haya un 2.
- ¿En qué parte de la memoria RAM está la variable i y sum? ¿Por qué en esas posiciones?  
R/ sum esta en 0, i en 1, estan ahi porque asi lo dicen los comentarios (indicaciones que se pueden dar en c++, mas no en lenguaje asemblador), y para permitir facilidad operandolos.
- Optimiza esta parte del código para que use solo dos instrucciones:
```
// i = i + 1
@i
D=M+1
@i
M=D
```  
R/
```
// i = i + 1
@i
M=M+1
```

### 10) Las posiciones de memoria RAM de 0 a 15 tienen los nombres simbólico R0 a R15. Escribe un programa en lenguaje ensamblador que guarde en R1 la operación 2 * R0.

```
@R0
D=M
D=D+M
M=D
```