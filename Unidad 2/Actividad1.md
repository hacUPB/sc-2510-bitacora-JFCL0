### Actividad 1

En el capítulo 4 del libro “The Elements of Computing Systems” que puedes encontrar [**aquí**](https://www.nand2tetris.org/_files/ugd/44046b_7ef1c00a714c46768f08c459a6cab45a.pdf), vas a repasar de nuevo cómo se realizan las operaciones de entrada y salida en la plataforma de cómputo que estamos estudiando, es decir, la plataforma Hack y la CPU Hack. También puedes ver [**este**](https://youtu.be/gTOFd80QfBU?si=6FLpT907cx1Q_NDB) video, si quieres, donde te explican el concepto. En la sección 4.2.5. vas a encontrar el concepto de entrada-salida mapeada a memoria o *memory maped* I/O. Analiza lo siguiente:

- ¿Qué es la entrada-salida mapeada a memoria?  
R/ Se refiere a la conexion de un teclado (entrada) y/o una pantalla (salida).  

- ¿Cómo se implementa en la plataforma Hack?  
R/ Intercambiando los valores asignados a las casillas de memoria RAM asignadas a la pantalla(1,0) y el teclado (valores referentes a cada tecla, preasignados) respectivamente.  

- Inventa un programa que haga uso de la entrada-salida mapeada a memoria.  
R/ 
```js
	@24576 //Uso de entrada, casilla del teclado.
	D=M
	@100 //el valor de la tecla "d" es de 100
	D=D-A
	@0
	D;JNE //Si D es diferente de 0 tras haber restado 100, sea positivo o negativo, se 
	reinicia el codigo.
	//Codigo HACK del dibujo, uso de salida (pantalla).
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
- Investiga el funcionamiento del programa con el simulador.  
R/ 👍
