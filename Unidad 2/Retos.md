## **Reto**

Ahora te propondré de nuevo unos mini-retos que te acercarán a la programación en alto nivel, pero desde el lenguaje ensamblador, precisamente para abordar la gran pregunta de esta unidad: ¿Cómo se pueden implementar en lenguaje ensamblador algunos conceptos básicos de programación en alto nivel?

¿Recuerdas los resultados de aprendizaje específicos (RAE) de este curso?

- RAE1: construyo aplicaciones interactivas aplicando patrones y estrategias que permitan alcanzar los requisitos funcionales y no funcionales establecidos. Se espera que llegues a un nivel resolutivo.
- RAE2: aplico pruebas de las partes y del todo de un software siguiendo metodologías, técnicas y estándares de la industria para garantizar el correcto funcionamiento de las aplicaciones. Se espera que llegues a un nivel autónomo.

El RAE1 lo evidenciarás con la construcción de las aplicaciones que proponen los retos, siguiendo los requisitos. El RAE2 lo evidenciarás con la implementación de pruebas utilizando el simulador.

<aside>
🔥

**Nota: Para la bitácora**
En la bitácora vas a reportar para cada mini-reto dos cosas:

- El código con la implementación (RAE1).
- Explica cómo probaste las partes del programa y cómo probaste el programa completo (RAE2).
</aside>

1. Escribe un programa en lenguaje ensamblador que sume los primeros 100 números naturales.
    
    ```cpp
    int i = 1;
    int sum = 0;
    While (i <= 100){
       sum += i;
       i++;
    }
    ```

    R/  

    ```cpp
    @1 //Variable i, contador
    @0 //Variable sum
    @0
    D=M
    @1
    M=M+1
    A=M
    D=D+A
    @0
    M=D
    @1
    D=M
    @100
    D=D-A
    @0
    D;JGT
    ```
    - ¿Cómo están implementadas las variables `i` y `sum`?  
    R/ i sirve como contador, sum sirve para mantener registro del progreso de la sumatoria.  

    - ¿En qué direcciones de memoria están estas variables?  
    R/ En el codigo inicial, los ubica en R0 y R1.  

    - ¿Cómo está implementado el ciclo `while`?  
    R/ Con un D mayor a 0 mientras almacena i-100, es decir, su valor maximo.  

    - ¿Cómo se implementa la variable `i`?  
    R? Dedicando un espacio de memoria a su almacenamiento.  
    - ¿En qué parte de la memoria se almacena la variable `i`?  
    R/ En el espacio que el usuario le de.  
    - Después de todo lo que has hecho, ¿Qué es entonces una variable?  
    R/ Un espacio almacenado de memoria.  
    - ¿Qué es la dirección de una variable?  
    R/ La que se le asigne.  
    - ¿Qué es el contenido de una variable?  
    R/ La que el usuario desee darle, o en otras palabras, M.  


2. Transforma el programa en alto nivel anterior para que utilice un ciclo `for` en vez de un ciclo `while`.  
    R/ 
    ```cpp
    int sum = 0;
    for (int i = 100; i > 0; i--)
    {
        sum += i;
    }
    ```
3. Escribe un programa en lenguaje ensamblador que implemente el programa anterior.
    R/
    ```cpp
    @100 //i=100
    M=A //Almacenar valor de i inicial
    @0 //sum = 0
    D=M
    @100 //i
    A=M
    D=D+A
    @0 //sum
    M=D
    @100 //i
    M=M-1 //i--
    D=M
    @2
    D;JNE //Loop donde si i llega a 0 o se vuelve negativo se detiene el programa.
    ```
4. Ahora vamos a acercarnos al concepto de puntero. Un puntero es una variable que almacena la dirección de memoria de otra variable. Observa el siguiente programa escrito en C++:
    
    ```cpp
    int a = 10;
    int *p;
    p = &a;
    *p = 20;
    ```
    
    El programa anterior modifica el contenido de la variable `a` por medio de la variable `p`. `p` es un puntero porque almacena la dirección de memoria de la variable `a`. En este caso el valor de la variable `a` será 20 luego de ejecutar `*p = 20;`. Ahora analiza:
    
    - ¿Cómo se declara un puntero en C++? `int *p;`. `p` es una variable que almacenará la dirección de un variable que almacena enteros.  
    R/ Al momento de declarar `p` se define una direccion en la memoria. Cuando se declara `*p`, de define el contenido o M de esta direccion. Es mediante `*p` que se puede modificar el valor de `a`. El asterisco hace referencia al contenido de la posicion de memoria.

    - ¿Cómo se define un puntero en C++? `p = &a;`. Definir el puntero es inicializar el valor del puntero, es decir, guardar la dirección de una variable. En este caso `p` contendrá la dirección de `a`.  
    R/ El puntero se define mediante poner el simbolo `&` frente a la direccion de RAM que se quiere almacenar en la otra variable.  

    - ¿Cómo se almacena en C++ la dirección de memoria de una variable? Con el operador `&`. `p = &a;`  
    R/ Efectivamente, con eso se hace.  

    - ¿Cómo se escribe el contenido de la variable a la que apunta un puntero? Con el operador ``. `p = 20;`. En este caso como `p` contiene la dirección de `a` entonces `p` a la izquierda del igual indica que quieres actualizar el valor de la variable `a`.  
    R/ Ya que `p` contiene la direccion, es mediante el uso del asterisco, ya que este se refiere a la posicion de `a`, esta es la forma de acceder facilmente al contenido de memoria de `a`.  
    
5. Traduce este programa a lenguaje ensamblador:
    
    ```cpp
    int a = 10;
    int *p;
    p = &a;
    *p = 20;
    ```
    R/
    ```cpp
    @10 //a=10, a esta ubicada en R10
    D=A //Almacenar direccion de a.
    @p  //Posicion no determinada, placeholder.
    M=D //Almacenar *p en p. R(p) = RAM(a)
    @20
    D=A
    @p
    A=M //Ir hacia p* / a.
    M=D //Almacenar 20 en a.
    ```
6. Ahora vas a usar un puntero para leer la posición de memoria a la que este apunta, es decir, vas a leer por medio del puntero la variable cuya dirección está almacenada en él.
    
    ```cpp
    int a = 10;
    int b = 5;
    int *p;
    p = &a;
    b = *p;
    ```
    
    En este caso `b = *p;` hace que el valor de `b` cambie de 5 a 10 porque `p` apunta a la la variable `a` y con `*p` a la derecha del igual estás leyendo el contenido de la variable apuntada.
    
7. Traduce este programa a lenguaje ensamblador:
    
    ```cpp
    int a = 10;
    int b = 5;
    int *p;
    p = &a;
    b = *p;
    ```
    R/
    ```cpp
    @10 // a = 10
    @5  // b = 5
    @*p // generar p.
    @10
    D=A
    @p
    M=D //p = &a, direccion de a queda almacenada.
    //Para hacer que b sea igual a *p, cambiaria la direccion de b, haciendo que esta este en la misma ubicacion que a.
    ```
8. Vas a parar un momento y tratarás de recodar de memoria lo siguiente. Luego verifica con un compañero o con el profesor.

    - ¿Qué hace esto `int *pvar;`?  
    R/ Crear una variable de puntero a otro, importa mas que por su direccion es por el valor que almacena.
    - ¿Qué hace esto `pvar = var;`?  
    R/ Hace que la direccion de 2 variables, pvar y var, sean la misma.
    - ¿Qué hace esto `var2 = *pvar`?  
    R/ Hace que el valor de var2 sea el que almacena pvar.
    - ¿Qué hace esto `pvar = &var3`?  
    R/ Hace que pvar almacene la direccion de var3.

9. Considera que el punto de entrada del siguiente programa es la función `main`, es decir, el programa inicia llamando la función `main`. Vas a proponer una posible traducción a lenguaje ensamblador de la función `suma`, cómo llamar a suma y cómo regresar a `std::cout << "El valor de c es: " << c << std::endl;` una vez suma termine.

    ```cpp
    #include <iostream>

    int suma(int a, int b) {
       int var = a + b;
       return var;
    }

    
    int main() {
       int c = suma(6, 9);
       std::cout << "El valor de c es: " << c << std::endl;
       return 0;
    }
    ```
    R/
    ```cpp
    @0 //Aqui voy a almacenar var.
    @a //Variable a.
    @b //Variable b.

    //suma
    @a
    D=M //Sumar valor almacenado en a
    @b
    D=D+M  //Sumar valor almacenado en b.
    @0     //var
    M=A    //Almacenar valor asignado a  var.
    @main //llevar a print
    D;JMP //forzar salto a los pasos correspondientes.

    //main
    @*a   //Llevar a asignar valor de a.
    D=A   //Almacenar valor de *a en D.
    @a    //Ir a valor asignado.
    M=D   //Almacenar valor asignado en su respectiva posicion. 
    @*b   //Llevar a asignar valor de b.
    D=A   //Almacenar valor de *b en D.
    @b    //Ir a valor asignado.
    M=D   //Almacenar valor asignado en su respectiva posicion.
    //Inserte aqui cualquiera que sea la forma de imprimir.
    @suma //Volver al inicio del codigo.
    D;JMP //Forzar salto tras completar accion.
    ```