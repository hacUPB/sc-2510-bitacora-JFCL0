### Actividad 4

Ahora realizarás una nueva variación al programa de la actividad anterior. Si se presiona la letra “d” muestras la imagen que diseñaste en el reto 18. Solo si se presiona la letra “e” borrarás la imagen que se muestra en pantalla.

R/  

Codigo original:
```
    //Limpiar pantalla
    @SCREEN
	D=A
	@R12
	AD=D+M
    //Programa original
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

Codigo modificado:

```
    //Programa original
    @24576 //Uso de entrada, casilla del teclado.
	D=M
	@100 //el valor de la tecla "d" es de 100
	D=D-A
	@0
	D;JNE //Si D es diferente de 0 tras haber restado 100, sea positivo o negativo, se reinicia el codigo.
    @24576 //Uso de entrada, casilla de teclado.
    D=M
	@101 //el valor de la tecla "d" es de 100
	D=D-A
	@61
	D;JEQ //Si D es igual a 0 tras haber restado 101, salta al codigo correspondiente de borrar lo que hay en pantalla.
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
    //Limpiar pantalla
    @SCREEN
	D=A
	@R12
	AD=D+M
	// return
	@R13
	A=M
	D;JMP

```