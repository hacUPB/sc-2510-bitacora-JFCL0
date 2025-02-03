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

### 11) Considera el siguiente programa:

```
i = 1000
LOOP:
if (i == 0) goto CONT
i = i - 1
goto LOOP
CONT:
```

La traducción a lenguaje ensamblador del programa anterior es:

```
// i = 1000
@1000
D=A
@i
M=D
(LOOP)
// if (i == 0) goto CONT
@i
D=M
@CONT
D;JEQ
// i = i - 1
@i
M=M-1
// goto LOOP
@LOOP
0;JMP
(CONT)
```
- ¿Qué hace este programa?  
R/ Este programa hace un loop un total de 1000 veces.

- ¿En qué memoria está almacenada la variable i? ¿En qué dirección de esa memoria?  
R/ En la RAM, posicion 1000.

- ¿En qué memoria y en qué dirección de memoria está almacenado el comentario //`i = 1000?`  
R/ ROM, posicion 0.

- ¿Cuál es la primera instrucción del programa anterior? ¿En qué memoria y en qué dirección de memoria está almacenada esa instrucción?  
R/ La primera instruccion es "@1000", esta almacenada en la ROM, posicion 1.

- ¿Qué son CONT y LOOP?  
R/ CONT y LOOP son posiciones en las instrucciones, es decir, en la ROM.

- ¿Cuál es la diferencia entre los símbolos `i` y `CONT`?  
R/ **i** es una variable, o al menos se trata de esta manera. **CONT** es una posicion en el codigo, especificamente, conocido como "Gate" (puerta).

### 12) Implemente en ensamblador:

```
R4 = R1 + R2 + 69
```
Sln

```
//Los valores son irrelevantes para el ejercicio
@R1
D=M
@69
M=D 
@R2 
D=M 
@69
D=D+M 
D=D+A 
// R4 = D 
```

### 13) Implemente en ensamblador:

```
if R0 >= 0 then R1 = 1
else R1 = –1

(LOOP)
goto LOOP
```

Sln

```
// if R0 >= 0 then R1 = 1; else R1=-1
@R0 
D=M
@R1
@9
D;JLT
M=M+1
@10
D;JGE
M=M-1

//LOOP; goto LOOP

(LOOP)
@LOOP
0;JMP
```
### 14) Implemente en ensamblador:

```
R4 = RAM(R1)
```

Sln

```
@R1
D=M
@R4 
M=D 
```

### 15) Implementa en ensamblador el siguiente problema. En la posición R0 está almacenada la dirección inicial de una región de memoria. En la posición R1 está almacenado el tamaño de la región de memoria. Almacena un -1 en esa región de memoria.
  
```
@R1
D=M //Guardar tamaño de la region
@R0 // Buscar puesto a asignar en la region
A=M // Ir hacia el siguiente puesto a asignar dentro de la region
M=-1 // Asignar valor en la region
@R0 // Ir hacia R0
M=M+1 // Aumentar en 1 para poder ir hacia el siguiente puesto a asignar en la region
D=D-1 // Reducir en conteo para permitir limitar el loop segun el tamaño asignado a la region
@2
D;JGT // Reiniciar el loop si todavia queda espacio en el tamaño asignado
```

### 16) Implementa en lenguaje ensamblador el siguiente programa:

```
int[] arr = new int[10];
int sum = 0;
for (int j = 0; j < 10; j++) {
    sum = sum + arr[j];
}
```

Sln

```
// R11 sera utilizado para salvaguardar el tamaño del array (10) y descontar por cada paso que se tome. R12 sera utilizado para mantener el valor de "sum". R13 sera utilizado para saber que casilla de la RAM sigue por sumar.

@R13 // Ir a la casilla de la RAM que sigue
A=M
D=M // D se lleva el valor dentro de la casilla.

@R12 // Se va a hacer la suma del valor dentro de la casilla seleccionada previamente.
D=D+M 
M=D

@R11 //Se va a reducir el conteo y determinar si se debe continuar. (j++)
M=M-1
D=M // D se lleva el valor del conteo para permitir el LOOP.

@R13 
M=M+1 // Se incrementa el valor en R13 para poder pasar a la siguiente casilla de suma.

@0
D;JGT // El LOOP, que se lleva a cabo a partir del conteo en R11, valor que D se llevo.
```
- ¿Qué hace este programa?  
R/ Este programa hace que la variable **sum** vaya agregando, uno a uno, los valores guardados dentro del array.

- ¿Cuál es la dirección base de arr en la memoria RAM?  
R/ Realmente no tengo ni idea, pero se me ocurre, desde R0 hasta donde indique el rango del array.

- ¿Cuál es la dirección base de sum en la memoria RAM y por qué?  
R/ Probablemente en las casillas despues de RAM(15), entonces siendo la primera declarada, R16.

- ¿Cuál es la dirección base de j en la memoria RAM y por qué?  
R/ Siguiendo la misma logica de antes, R17.

### 17) Implementa en lenguaje ensamblador:

```
if ( (D - 7) == 0) goto a la instrucción en ROM[69]
```
Sln

```
@69
M=A // Profe, por que tanto 69, me estoy enloqueciendo, a donde mire, esta ese numero :c.
@7
D=D-A
@69
D;JEQ
```

### 18) Utiliza esta (link) herramienta para dibujar un bitmap en la pantalla.

![I'm gonna cry now](https://cdn.discordapp.com/attachments/936102461968638072/1334998133406433290/Screenshot_2025-01-31_162218.png?ex=679e9166&is=679d3fe6&hm=bba97aa7afd8aaabf4e32dde8be481fc3f20fd5565ebf94726b1a43289c5497c&)

### 19) Analiza el siguiente programa escrito en lenguaje de maquina

```
0100000000000000
1110110000010000
0000000000010000
1110001100001000
0110000000000000
1111110000010000
0000000000010011
1110001100000101
0000000000010000
1111110000010000
0100000000000000
1110010011010000
0000000000000100
1110001100000110
0000000000010000
1111110010101000
1110101010001000
0000000000000100
1110101010000111
0000000000010000
1111110000010000
0110000000000000
1110010011010000
0000000000000100
1110001100000011
0000000000010000
1111110000100000
1110111010001000
0000000000010000
1111110111001000
0000000000000100
1110101010000111
```

Traduccion

```
@16384
D=A
@16
M=D
@24576
D=M
@19
D;JNE
@4
D;JLE
@16
AM=M-1
M=0
@4
0;JMP
@16
D=M
@24576
D=D-A
@4
D;JGE
@16
M=M+1
@4
0;JMP
```
- ¿Qué hace este programa?  
R/ Darme ganas de morir, fuera de eso...  
[0] Ir a RAM(16384)  
[1] Hacer que D = 16384  
[2] Ir a RAM(16)  
[3] Hacer que RAM(16) = 16384  
[4] Ir a RAM(24576)  
[5] Hacer que D = 0  
[6] Ir a RAM(19)  
[7] Si D es != de 0, ir a ROM(19)  
[8] Ir a RAM(16)  
[9] Hacer que D = 16384  
[10] Ir hacia RAM(16384)  
[11] D = 16384 - 16384 = 0; hacer que D = 0  
[12] Ir hacia RAM(4)  
[13] Saltar a ROM(4) si D es menor o igual a 0.  
[>=14] El programa se queda en un bucle, asi que nos quedaremos sin saber.

### 20) Implementa un programa en lenguaje ensamblador que dibuje el bitmap que diseñaste en la pantalla solo si se presiona la tecla “d”.

```
	@24576 //Casilla del teclado.
	D=M
	@100 //el valor de la tecla "d" es de 100
	D=D-A
	@0
	D;JNE //Si D es diferente de 0 tras haber restado 100, sea positivo o negativo, se reinicia el codigo.
	//Codigo HACK del dibujo.
	// put bitmap location value in R12
	// put code return address in R13
	@SCREEN
	D=A
	@R12
	AD=D+M
	// row 2
	@231 // A holds val
	D=D+A // D = addr + val
	A=D-A // A=addr + val - val = addr
	M=D-A // RAM[addr] = val
	// row 3
	D=A // D holds previous addr
	@32
	AD=D+A
	@161 // A holds val
	D=D+A // D = addr + val
	A=D-A // A=addr + val - val = addr
	M=D-A // RAM[addr] = val
	// row 4
	D=A // D holds previous addr
	@32
	AD=D+A
	@231 // A holds val
	D=D+A // D = addr + val
	A=D-A // A=addr + val - val = addr
	M=D-A // RAM[addr] = val
	// row 5
	D=A // D holds previous addr
	@32
	AD=D+A
	@133 // A holds val
	D=D+A // D = addr + val
	A=D-A // A=addr + val - val = addr
	M=D-A // RAM[addr] = val
	// row 6
	D=A // D holds previous addr
	@32
	AD=D+A
	@231 // A holds val
	D=D+A // D = addr + val
	A=D-A // A=addr + val - val = addr
	M=D-A // RAM[addr] = val
	// return
	@R13
	A=M
	D;JMP
```
