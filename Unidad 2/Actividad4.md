### Actividad 4

Ahora realizarás una nueva variación al programa de la actividad anterior. Si se presiona la letra “d” muestras la imagen que diseñaste en el reto 18. Solo si se presiona la letra “e” borrarás la imagen que se muestra en pantalla.

R/  

Codigo original:
```js
	//Programa original
	@INICIO
    	@24576 //Uso de entrada, casilla del teclado.
	D=M
	@100 //el valor de la tecla "d" es de 100
	D=D-A
	@BITMAP //No quiero contar
	D;JEQ //Si D es igual a 0 tras haber restado 100, pasa a hacer el dibujo
	//Limpiar pantalla
	@SCREEN
	D=A
	@INICIO
	AD=D+M
	D;JMP
	//Codigo HACK del dibujo, uso de salida (pantalla).
	@SCREEN
	D=A
	@BITMAP
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
	@INICIO
	A=M
	D;JMP
```

Codigo modificado:

```
	//Programa original
	@INICIO
    	@24576 //Uso de entrada, casilla del teclado.
	D=M
	@100 //el valor de la tecla "d" es de 100
	D=D-A
	@BITMAP //No quiero contar
	D;JEQ //Si D es igual a 0 tras haber restado 100, pasa a hacer el dibujo
	//Si no paso el chequeo de presionar d, procede con el chequeo de si presiona e
	@24576 //Uso de entrada, casilla del teclado.
	D=M
	@101 //el valor de la tecla "e" es de 101
	D=D-A
	@BORRARDIBUJO //No quiero contar
	D;JEQ //Si D es igual a 0 tras haber restado 101, pasa a borrar el dibujo
	//Codigo HACK del dibujo, uso de salida (pantalla).
	@SCREEN
	D=A
	@BITMAP
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
	@INICIO
	A=M
	D;JMP
	@BORRARDIBUJO
	//Limpiar pantalla
	@SCREEN
	D=A
	@INICIO
	AD=D+M
	D;JMP
```
