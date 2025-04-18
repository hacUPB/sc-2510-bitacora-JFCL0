# Sesion 1: La naturaleza de los objetos en C++

## Introduccion a los objetos

Concepto: un objeto es una instancia de una clase que combina datos (atributos) y comportamiento (métodos).

Considera el siguiente caso de estudio:

```cpp
class Particle {
public:
    float x, y;
    void move(float dx, float dy) {
        x += dx;
        y += dy;
    }
};
```

### ¿Qué representa la clase Particle?  
La clase **Particle** representa a un tipo de dato, que almacena un valor de x, y, y tiene un metodo de mover, desde el cual se hace que a x se le sume el largo desde x del tamaño de la particula.
### ¿Cómo interactúan sus atributos y métodos?
En esta clase, primero se generan **atributos**, desde los cuales se crea y cuyos valores se almacenan en el objeto. Estos valores se utilizan en el unico **metodo** que tiene esta clase, desde  el cual se cambian sus valores.
### Prompt para ChatGPT: explícame en detalle qué es un objeto en C++ y cómo se relaciona con una clase. Usa el ejemplo de una clase Particle con atributos x y y y un método move.
En resumidas cuentas, la respuesta que dio la IA fue: Una clase es la forma en la que se establece un nuevo tipo de dato, que tiene distintos **atributos** y **metodos**. De la misma forma en que se declaran otros tipos de variable como `float` o `string`, al momento de crear un objeto se tiene primero que declarar que es el tipo de objeto que la clase especifica, y al momento de crearla, se declaran tambien los distintos valores que fueron declarados mediante sus **atributos**. Estos nuevos objetos tienen distintos **metodos** añadidos, es decir, pueden llamar a un **void** mediante una unica linea de codigo, y esta sera igual para cada objeto creado dentro de esta clase.

#### Ej:

```cpp
#include <iostream>
using namespace std;

// Definición de la clase Particle (el "plano")
class Particle {
public:
    // Atributos (variables miembro)
    float x;
    float y;
    
    // Método para mover la partícula
    void move(float dx, float dy) {
        x += dx;
        y += dy;
    }
    
    // Método para mostrar la posición
    void showPosition() {
        cout << "Posición: (" << x << ", " << y << ")" << endl;
    }
};

int main() {
    // Creación de objetos (instancias de la clase Particle)
    Particle particle1;  // Objeto 1
    Particle particle2;  // Objeto 2
    
    // Inicialización de los objetos
    particle1.x = 0.0;
    particle1.y = 0.0;
    
    particle2.x = 5.0;
    particle2.y = 3.0;
    
    // Usando métodos de los objetos
    cout << "Antes de mover:" << endl;
    particle1.showPosition();
    particle2.showPosition();
    
    particle1.move(2.5, 1.5);
    particle2.move(-1.0, 0.5);
    
    cout << "\nDespués de mover:" << endl;
    particle1.showPosition();
    particle2.showPosition();
    
    return 0;
}
```

## Explorando la memoria

Objetivo: entender cómo se almacenan los objetos en memoria.

Considera los siguientes pasos:

1. Crear instancias:

    ```cpp
    Particle p1;
    Particle p2;
    ```

2. Explorar la memoria usando punteros:

    ```cpp
    std::cout << "Dirección de p1: " << &p1 << std::endl;
    std::cout << "Dirección de p2: " << &p2 << std::endl;
    ```

3. Determinar el tamaño del objeto:

    ```cpp
    std::cout << "Tamaño de Particle: " << sizeof(Particle) << " bytes" << std::endl;
    ```

4. Imprimir direcciones de atributos:

    ```cpp
    std::cout << "Dirección de p1.x: " << &(p1.x) << std::endl;
    std::cout << "Dirección de p1.y: " << &(p1.y) << std::endl;
    ```

Reflexión Guiada:

### ¿Los atributos están almacenados de forma contigua?

Si, la totalidad de almacenamiento de cada objeto `Particle` es de 8 bytes, siendo 4 dedicados a `x` y 4 dedicados a `y`, por lo que su almacenamiento se da de forma contigua, es decir, sin `padding` o espacios de almacenamiento en RAM de intermedio.
### ¿Qué indica el tamaño del objeto sobre su estructura interna?
Que los metodos del objeto no ocupan espacio de almacenamiento en RAM. 
### Prompt para ChatGPT: ¿Cómo se almacenan los objetos en memoria en C++? Si tengo dos instancias de Particle, ¿Cómo se relacionan sus direcciones de memoria? ¿Los atributos están contiguos?
En resumidas cuentas, estas fueron las conclusiones de la IA:

- Los atributos estan contiguos, es decir, `y` sigue inmediatamente despues de `x`.
- La direccion de `&p1` es igual a la de su primer atributo, `&p1.x`.
- Cada objeto tiene su propio espacio de RAM dedicado, es decir, las casillas de RAM dedicadas a `p1` no se sobreponen con las de `p2`. Esto a su vez significa que cada objeto tiene sus valores de `x` y `y` independientes.
- Los objetos no necesariamente se almacenaran de forma **contigua**, es decir, depende del tipo de memoria que tengan dedicados. Por ejemplo, de estar almacenados en el **stack**, podrian tener otras variables requeridas por el programa de por medio, y en el caso de ser almacenados en el **heap** dependeria enteramente del **asignador a memoria dinamica**. 

## Análisis de diferencias

Objetivo: comparar cómo diferentes clases afectan el uso de memoria.

Considera los siguientes pasos:

1. Crear clases con diferentes atributos y métodos:

    ```cpp
    class Simple {
    public:
        int a;
    };

    class Complex {
    public:
        int a, b, c;
        void method1() {}
        void method2() {}
    };
    ```

2. Compara los tamaños:

    ```cpp
    std::cout << "Tamaño de Simple: " << sizeof(Simple) << " bytes" << std::endl;
    std::cout << "Tamaño de Complex: " << sizeof(Complex) << " bytes" << std::endl;
    ```

3. Agregar datos estáticos y dinámicos:

    ```cpp
    class StaticData {
    public:
        static int s;
        int a;
    };

    int StaticData::s = 0;

    class DynamicData {
    public:
        int* ptr;
        DynamicData() {
            ptr = new int[10];
        }
        ~DynamicData() {
            delete[] ptr;
        }
    };
    ```

### ¿Cómo afectan los datos estáticos al tamaño de la instancia?
Al momento de crear un objeto y marcarlo como `static`, su valor sera el mismo para **todos** los objetos creados **bajo esa clase**. Debido a esto, el valor asignado a este atributo no sera almacenado en consonancia con el resto del objeto, sino que sera almacenado en la memoria **global**. Debido a esto, los valores y variables marcados como `static` no contribuiran a el tamaño del objeto.
### ¿Qué diferencias hay entre datos estáticos y dinámicos en términos de memoria?
Cuando un objeto es almacenado en la **memoria global** este se inicializa una sola vez, es eliminado automaticamente tan pronto el programa se finaliza. Estos tienen distintos comportamientos dependiendo de en donde sean creados: 
- En caso de crearse en el programa principal, estas se mantienen y se permite acceder a ellas desde su totalidad.
- En caso de crearse al interior de una clase, es decir, como uno de sus atributos, este se mantendra a traves de todos los objetos pertenecientes a esa clase.
- En caso de crearse al interior de una funcion, su valor podra llamarse en cualquier otra funcion, pero solamente se podra editar dentro de la misma funcion en la que fue creado.

Cuando un objeto es almacenado en la memoria **dinamica** (heap) estos solamente pueden ser editados mediante el uso de **punteros**, generalmente requieren de un metodo adicional que permita eliminarlas (de lo contrario, se puede dar una fuga de memoria), y su gestion en su totalidad queda en manos del programador, ya que estas permanecen incluso despues de que el programa fue finalizado.
### Prompt para ChatGPT: explícame cómo el uso de variables estáticas y dinámicas en una clase afecta el tamaño de sus instancias. ¿Las variables estáticas ocupan espacio en cada objeto?

---

## **Tabla Comparativa**
| **Aspecto**            | **Datos Estáticos (`static`)**                          | **Datos Dinámicos (`new`/`malloc`)**               |
|------------------------|-------------------------------------------------------|---------------------------------------------------|
| **Almacenamiento**     | Memoria estática (segmento de datos global)           | Heap (memoria dinámica)                           |
| **Vida útil**          | Toda la ejecución del programa                        | Hasta que se libere con `delete`/`free`           |
| **Ámbito**             | Global o compartido entre instancias (si es `static` en una clase) | Limitado al puntero/referencia que los apunta |
| **Inicialización**     | Automática (antes de `main()`)                        | Manual (en tiempo de ejecución)                   |
| **Gestión de memoria** | Automática (liberada al terminar el programa)         | Manual (responsabilidad del programador)          |
| **Rendimiento**        | Acceso rápido (dirección fija en memoria)             | Acceso más lento (indirección por puntero)        |
| **Uso típico**         | Contadores globales, configuraciones compartidas      | Estructuras de datos dinámicas, objetos grandes   |

---

## Reflexión

Documenta en tu bitácora de aprendizaje:

### ¿Qué es un objeto desde la perspectiva de la memoria?
Un objeto es un rango de espacios en la memoria RAM que almacenan **atributos**.
### ¿Cómo influyen los atributos y métodos en el tamaño y estructura del objeto?
Los **atributos** son ubicados en los espacios de memoria RAM **dedicados al objeto**. Los **metodos** **no son almacenados en la memoria RAM dedicada al objeto**.
### Conclusión: resumir los hallazgos y cómo esto impacta el diseño de clases.
Al momento de crear un objeto, si se requiere que este este almacenado en un **array** que exista fuera de la funcion mediante la que fue creada, es ideal almacenarlo en la memoria **heap**, sin embargo, no se debe olvidar crear un metodo que lo elimine dentro del mismo programa, con el fin de que esta no se quede ocupando espacio en la memoria del equipo. Al momento de crear valores que sean consistentes a traves de todo el programa, y que se busque que no puedan editarse mediante cualquier funcion, es ideal almacenarlo en la memoria **global**.

# Sesión 2: ubicación en memoria de datos y métodos

Objetivo: explorar cómo y dónde se almacenan los datos y métodos de una clase en la memoria del computador.

## Análisis de la estructura de una clase

- Datos: los atributos se almacenan en el stack o heap, según cómo se instancien.
- Métodos: el código de los métodos reside en la sección de código (texto) del programa.
- Vtables: estructuras usadas para implementar métodos virtuales.

### Prompt para ChatGPT: ¿Dónde se almacenan los datos y métodos de una clase en C++ en la memoria? Explica el concepto de vtable y cómo se relaciona con los métodos virtuales.

Los **atributos** de una clase son almacenados en la memoria **stack** o **heap** dependiendo de como se declare el objeto, a no ser que sean **static**, en ese caso, se almacenan en la **memoria global**.  
Los **metodos** no virtuales son almacenados en el mismo area en el que se almacena el resto del codigo, especificamente, como **funciones globales**. Los metodos **virtuales** (vtable) se refieren a un **array de punteros**, donde cada objeto tiene un puntero oculto a esta tabla, siendo unicamente 1 por **clase** y permiten polimorfismo; esta tabla es almacenada en la **memoria estatica**, agregando 8 bytes adicionales de almacenamiento a cada objeto creado bajo la clase, destinados al **puntero** que lleva a la tabla, y requieren de un metodo adicional para ser eliminados. 

## Exploración de métodos virtuales

Objetivo: comprender el impacto de los métodos virtuales en la memoria y el rendimiento.

Considera los siguientes pasos:

1. Crea una jerarquía de clases con métodos virtuales:

    ```cpp
    class Base {
    public:
        virtual void display() {
            std::cout << "Base display" << std::endl;
        }
    };

    class Derived : public Base {
    public:
        void display() override {
            std::cout << "Derived display" << std::endl;
        }
    };
    ```

2. Analiza la vtables. ¿En qué parte de la memoria se encuentran las vtable de cada objeto?

    ```cpp
    Base b;
    Derived d;
    std::cout << "Vtable de Base: " << *(void**)&b << std::endl;
    std::cout << "Vtable de Derived: " << *(void**)&d << std::endl;
    ```

3. Usar un depurador: observar cómo se resuelven las llamadas a métodos virtuales en tiempo de ejecución. Puedes usar para ello varias herramientas del depurador. En el menú Debug de Visual Studio, selecciona Windows y luego Memory. Allí podrás explorar la memoria. Si das click derecho sobre una función del código puedes ver el programa en ensamblador con Go to Dissasembly.
### ¿Cómo afecta la presencia de métodos virtuales al tamaño del objeto?
La presencia de metodos virtuales afecta al tamaño del objeto ya que cada objeto creado tendra 8 bytes adicionales de memomoria dedicados al **puntero** conectado con la `vtable`.
### ¿Qué papel juegan las vtables en el polimorfismo?
Este tipo de metodos permiten **heredar** un metodo a la clase derivada, y que este pueda ser editado mediante la clase derivada, es decir, que al momento de llamar el metodo desde la clase principal, **todos los objetos creados en clases derivadas** ejecuten su propia version del metodo original.
### Prompt para ChatGPT: ¿Cómo se implementan los métodos virtuales en C++? Explica el concepto de vtable y cómo se utiliza para resolver llamadas a métodos virtuales.

En resumidas cuentas, lo que dice la IA es: Los metodos virtuales son aquellos que permiten implementar polimorfismo en C++.  

Para poder declarar un metodo virtual se debe iniciar con `virtual void metodo() {}`, digamos estamos creando esto en una clase **inicial**, declarada con `class inicial`. A partir de esto, al momento de crear una clase derivada, digamos **derivada** se hace con `class derivada::public inicial`, y tras esto, para editar el metodo virtual, se debe hacer de la siguiente manera: `void metodo() override {}`.

Ej:

```cpp
class Animal {
public:
    virtual void hablar() { cout << "Sonido genérico" << endl; }
    virtual ~Animal() {}  // Destructor virtual (¡importante!)
};

class Perro : public Animal {
public:
    void hablar() override { cout << "Guau!" << endl; }
};
```

Ahora, al haber creado la `vtable`, cada objeto creado bajo una clase con un **metodo virtual** tendra un puntero asignado al interior de la `vtable`, de forma que al momento de llamar el metodo general, el programa ira por cada elemento de la tabla, detectando cada objeto, detectara la funcion que le esta pidiendo y la ejecutara activando el **metodo derivado** del metodo virtual original.

Ej:

```cpp
Animal* animal = new Perro();
animal->hablar();  // Llama a Perro::hablar() gracias a la vtable
delete animal;     // Llama al destructor correcto
```

## Uso de punteros y referencias

Objetivo: explorar cómo los punteros y referencias afectan la gestión de la memoria y la llamada a métodos.

Considera estos pasos:

### 1. Implementar una clase con punteros a funciones:

```cpp
class FunctionPointerExample {
public:
    void (*funcPtr)();
    static void staticFunction() {
        std::cout << "Static function called" << std::endl;
    }
    void assignFunction() {
        funcPtr = staticFunction;
    }
};
```

### 2. Analizar el impacto en memoria:
- Observar si el tamaño de la instancia cambia al usar punteros a funciones.  
Si, este incrementa por 8 bytes, siendo esto lo que regularmente ocupa un puntero, incluidos los que apuntan a funciones, siendo este el caso.
- Verificar cómo se almacenan estos punteros.  
`tipo_retorno (*nombre_puntero)(tipo_param1, tipo_param2, ...);`
### 3. Reflexión Guiada:
- ¿Cuál es la relación entre los punteros a métodos y la vtable?  
La `vtable` es un array de **punteros a funciones**.
- ¿Cómo afectan estos mecanismos al rendimiento del programa?  
Este tipo de mecanismos vienen con su precio, es decir, al utilizarlos se esta sacrificando la capacidad de **optimizar el programa**. Al utilizar estos metodos la CPU debe leer la direccion almacenada en el puntero, para despues saltar a esa direccion y ejecutar su contenido. Al utilizar `vtables` se deben dedicar 8 bytes mas de RAM para cada objeto creado, y a su vez, al momento de llamar un **metodo virtual** el programa debe acceder al **puntero**, buscar la direccion del metodo en la tabla y saltar a esa direccion, gastando mas recursos para poder ejecutarse.
- Prompt para ChatGPT: ¿Qué diferencia hay entre punteros a funciones y punteros a métodos miembro en C++? ¿Cómo afectan al tamaño de los objetos y al rendimiento?  

---

#### **1. Definición y Contexto**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| Apuntan a funciones globales o estáticas.    | Apuntan a funciones miembro no estáticas de una clase. |
| No están asociados a un objeto.              | Requieren un objeto para ser invocados (tienen contexto `this`). |


---

#### **2. Sintaxis**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| `tipo_retorno (*nombre)(parámetros);`       | `tipo_retorno (Clase::*nombre)(parámetros);`  |
| Asignación: `nombre = &función;`            | Asignación: `nombre = &Clase::método;`       |


---

#### **3. Invocación**
| **Punteros a Funciones**                     | **Punteros a Métodos**                         |
|---------------------------------------------|-----------------------------------------------|
| Llamada directa: `ptrFunc(args);`           | Requiere un objeto: `(objeto.*ptrMetodo)(args);` o `(ptrObjeto->*ptrMetodo)(args);` |


---

#### **4. Almacenamiento del Contexto (`this`)**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| No manejan contexto (`this` no existe).      | Almacenan implícitamente el contexto del objeto (`this`). |
|                                              | Por eso necesitan un objeto para ser llamados. |

---

#### **5. Compatibilidad con `std::function` y Lambdas**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| Pueden asignarse directamente a `std::function`. | Requieren `std::bind` o lambdas para usarse en `std::function`. |


---

#### **6. Rendimiento**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| Llamada más rápida (sin indirección de `this`). | Llamada ligeramente más lenta (manejo de `this`). |
| Ideal para callbacks simples.                | Útiles para diseño orientado a objetos.       |

---

#### **7. Casos de Uso Típicos**
| **Punteros a Funciones**                     | **Punteros a Métodos**                          |
|---------------------------------------------|-----------------------------------------------|
| - Callbacks en bibliotecas C (ej: `qsort`). | - Patrón Strategy en clases.                  |
| - Eventos en APIs de bajo nivel.            | - Implementación de señales/slots.            |

---
### 4. Reflexión individual:
- ¿Dónde residen los datos y métodos de una clase en la memoria?  
Los datos de una clase residen en la memoria **stack** o **dinamica** dependiendo de donde se declare que deben estar, a no ser que sean declaradas como `static`, en cuyo caso, se almacenan en la memoria global. Los metodos de una clase, generalmente, se almacenan en la memoria **global**, a no ser que sean metodos **virtuales** en cuyo caso, cada objeto tendra un **puntero** añadido, y la `vtable` que se almacena en memoria **global** almacenara **punteros a funcion** por cada clase creada.
- ¿Cómo interactúan las diferentes partes en tiempo de ejecución?  
Las `vtables` y en consecuente, los metodos **virtuales**, y en consecuencia, los **punteros a funcion** incrementan la memoria que el programa debe utilizar, ya que agregan procesos adicionales que **no se pueden optimizar**, incrementando asi el tiempo de ejecucion requerido para los programas.
- Conclusión: cómo esta comprensión afecta el diseño de sistemas.  
Entender estos conceptos permite la creacion de **clases derivadas**, sin embargo, tambien deja como pregunta los casos en los que el peso que dejan estas funciones vale la pena por las oportunidades que dan estas a hacer de un programa algo mas eficiente de escribir.

# Sesión 3: implementación Interna de Encapsulamiento, Herencia y Polimorfismo

Objetivo: entender cómo se implementan las abstracciones del encapsulamiento, herencia y polimorfismo a nivel interno.

## Profundizando en el encapsulamiento

Modificadores de acceso: private, protected, public. Control de acceso: implementado por el compilador, no por restricciones en tiempo de ejecución.

Crear una clase con diferentes niveles de acceso:

```cpp
class AccessControl {
private:
    int privateVar;
protected:
    int protectedVar;
public:
    int publicVar;
    AccessControl() : privateVar(1), protectedVar(2), publicVar(3) {}
};
```

Intentar acceder a los miembros desde una función externa:

```cpp
int main() {
    AccessControl ac;
    ac.publicVar = 10; // Válido
    // ac.protectedVar = 20; // Error de compilación
    // ac.privateVar = 30; // Error de compilación
    return 0;
}
```

> **Prompt para ChatGPT:** ¿Cómo implementa el compilador el encapsulamiento en C++? Si los miembros privados aún ocupan espacio en el objeto, ¿Qué impide que se acceda a ellos desde fuera de la clase?
>

Los marcadores de acceso afectan el tiempo de compilacion pero no el tiempo de ejecucion, esto ya que estos marcadores **no se envian en el codigo binario**, pero el compilador verifica que todo este correcto antes de hacer esta traduccion. Tras ser compilado, se pueden utilizar metodos para poder editar las variables privadas, sin embargo esto no es recomendado ya que se considera como **comportamiento indefinido**.

## Hagamos un poco de hackers y rompamos el encapsulamiento

Objetivo: el objetivo de este experimento es demostrar cómo se pueden violar las restricciones de acceso a los miembros privados de una clase en C++ utilizando técnicas de bajo nivel como la aritmética de punteros y reinterpret_cast. Se ilustrará cómo es posible acceder a los datos privados de un objeto desde fuera de la clase, lo que rompe el principio de encapsulamiento.

Considera los siguientes pasos:

1. Definición de la clase: primero, define una clase con miembros privados para observar el acceso a esos miembros desde fuera de la clase.

    ```cpp
    #include <iostream>

    class MyClass {
    private:
        int secret1;
        float secret2;
        char secret3;

    public:
        MyClass(int s1, float s2, char s3) : secret1(s1), secret2(s2), secret3(s3) {}

        void printMembers() const {
            std::cout << "secret1: " << secret1 << "\n";
            std::cout << "secret2: " << secret2 << "\n";
            std::cout << "secret3: " << secret3 << "\n";
        }
    };
    ```

2. Intento de acceso ilegal: verifica que no se puede acceder a los miembros privados directamente utilizando el compilador:

    ```cpp
    int main() {
        MyClass obj(42, 3.14f, 'A');
        // Esta línea causará un error de compilación
        std::cout << obj.secret1 << std::endl;

        obj.printMembers();  // Método público para mostrar los valores
        return 0;
    }
    ```

El código anterior intentará acceder directamente al miembro privado secret1, lo que resultará en un error de compilación.

3. Acceso a miembros privados usando reinterpret_cast: ahora, observa cómo violar el encapsulamiento utilizando reinterpret_cast y la aritmética de punteros:

    ```cpp
    int main() {
        MyClass obj(42, 3.14f, 'A');

        // Usando reinterpret_cast para violar el encapsulamiento
        int* ptrInt = reinterpret_cast<int*>(&obj);
        float* ptrFloat = reinterpret_cast<float*>(ptrInt + 1);
        char* ptrChar = reinterpret_cast<char*>(ptrFloat + 1);

        // Accediendo y mostrando los valores privados
        std::cout << "Accediendo directamente a los miembros privados:\n";
        std::cout << "secret1: " << *ptrInt << "\n";       // Accede a secret1
        std::cout << "secret2: " << *ptrFloat << "\n";     // Accede a secret2
        std::cout << "secret3: " << *ptrChar << "\n";      // Accede a secret3

        return 0;
    }
    ```

Ahora reflexiona:

- ¿Qué es el encapsulamiento y cuál es su propósito en la programación orientada a objetos?  
El proposito de esto es proteger y controlar el acceso a los datos de un objeto, con el fin de hacer de los cambios de estos en algo predecible y seguro.
- ¿Por qué es importante proteger los datos de una clase y restringir el acceso desde fuera de la misma?  
Mediante el uso de estas notaciones se puede **evitar modificaciones incorrectas** si se declara como `private`, **exponer solo lo necesario** si se declara como `public`. Tambien **facilitan el mantenimiento** del programa, **reducen el acoplamiento** y **dan soporte de abstraccion**, es decir, muestran al usuario solamente lo que tiene que ver, no el como se hace.

- ¿Qué significa reinterpret_cast y cómo afecta la seguridad del programa?  
Este operador compromete la seguridad del programa debido a que permite **reinterpretar arbitrariamente** los bits de un tipo de dato como si se trataran de otro tipo.
- ¿Por qué crees que se pudo acceder a los miembros privados de MyClass en este experimento, a pesar de que el compilador normalmente lo impediría?  
Debido al operador `reinterpret_cast`, el cual puede violar las medidas de encapsulamiento.
- ¿Cuáles podrían ser las consecuencias de utilizar técnicas como las mostradas en este experimento en un programa real?  
Puede resultar en **corromper la variable**, **generar fallos de ejecucion** y **atentando contra la portabilidad de un programa.** Al momento de tratar con punteros, estos pueden causar **errores de hardware** y **corrupcion de memoria**, al buscar datos en posiciones de memoria incorrectas. 
- ¿Qué implicaciones tiene este experimento sobre la confianza en las barreras de encapsulamiento que proporciona C++?  
Las barreras de encapsulamiento que proporciona C++ no son inquebrantables, sin embargo, este experimento permite evidenciar el **motivo** detras de estas barreras y el motivo de por que debe evitarse traspasarlas sin un motivo **bien justificado**.

## Herencia y la Relación en Memoria

- Layout de memoria: ¿Cómo los objetos derivados contienen los datos de las clases base?
- Desplazamiento en memoria: los atributos de la clase base preceden a los de la clase derivada.

Considera los siguientes pasos:

1. Crear clases con herencia:

```cpp
class Base {
public:
    int baseVar;
};
class Derived : public Base {
public:
    int derivedVar;
};
```

1. Analizar las direcciones de memoria:

```cpp
Derived d;
std::cout << "Dirección de d: " << &d << std::endl;
std::cout << "Dirección de d.baseVar: " << &(d.baseVar) << std::endl;
std::cout << "Dirección de d.derivedVar: " << &(d.derivedVar) << std::endl;
```

- ¿Cómo se organizan los atributos en memoria?  
Primero se ubican todos los miembros de la clase **base**, tras esto, se colocan los miembros de la clase **derivada**.

- ¿Qué sucede si agregamos más niveles de herencia?  
Cada miembro esta desplazado seguin el tamaño acumulativo de los miembros anterioriores, es decir, todavia sigue la misma teoria, donde la clase **mayor** va en el tope de los espacios dedicados, tras los espacios que ocupan sus componentes, iria la clase de en **medio**, y tras los espacios que ocupan sus componentes iria la clase **menor**.

Prompt para ChatGPT: explícame cómo se organiza en memoria un objeto de una clase derivada en C++. ¿Cómo se almacenan los atributos de la clase base y de la derivada?

| Nivel de Herencia | Clase | Miembro | Posición en Memoria | Offset Relativo |
|-------------------|-------|---------|---------------------|----------------|
| Base              | GrandParent | grandParentVar | Base + 0x00 | 0 bytes |
| Nivel 1           | Parent | parentVar | Base + 0x04 | 4 bytes |
| Nivel 2           | Child | childVar | Base + 0x08 | 8 bytes |

Notas:
* La dirección "Base" representa la dirección base del objeto completo (`Child c`).
* Los offsets están basados en `int` de 4 bytes (puede variar según la arquitectura).
* Cada nivel de herencia coloca sus miembros después de todos los miembros de sus ancestros.
* El objeto completo ocupa 12 bytes (3 variables int de 4 bytes cada una).

Representación Visual:

```
Dirección baja  +------------------+
Base + 0x00     | grandParentVar   |  GrandParent
                +------------------+
Base + 0x04     | parentVar        |  Parent
                +------------------+
Base + 0x08     | childVar         |  Child
                +------------------+
Dirección alta
```

## Polimorfismo y Vtables en detalle

- Polimorfismo: permite usar una interfaz común para diferentes tipos de objetos.
- Implementación: uso de vtables para resolver métodos virtuales.

Considera los siguientes pasos:

1. Crear una jerarquía polimórfica:

    ```cpp
    class Animal {
    public:
        virtual void makeSound() {
            std::cout << "Some generic sound" << std::endl;
        }
    };
    class Dog : public Animal {
    public:
        void makeSound() override {
            std::cout << "Bark" << std::endl;
        }
    };
    class Cat : public Animal {
    public:
        void makeSound() override {
            std::cout << "Meow" << std::endl;
        }
    };
    ```

2. Usar punteros de la clase base:

    ```cpp
    Animal* animals[] = { new Dog(), new Cat() };
    for (Animal* animal : animals) {
        animal->makeSound();
    }
    ```

3. Analizar con un depurador: observa cómo se resuelven las llamadas a makeSound() en tiempo de ejecución.
- ¿Cómo utiliza el programa las vtables para el polimorfismo?  
El camino que toma este programa es sencillo. `animal->makeSound()` > `_vptr` > `vtable` > `animal::makeSound()` > `vtable` > `Dog (class Dog)`/`Cat (class Cat)` > `Dog::makeSound()`/`Cat::makeSound`. Este camino, explicado, empieza por la llamada a la funcion, desde la cual se procede al `vtable`, en la cual lee tanto los objetos como la funcion que pide ejecutar, desde la cual la ejecuta en **ambos** objetos, los cuales tienen el **metodo modificado** por sus respectivas clases, llevando a que el Perro diga "Bark" y el Gato diga "Meow".

- ¿Cuál es el impacto en el rendimiento?  
El impacto en el rendimiento es el mismo que naturalmente tienen los **metodos virtuales**, sin embargo, al utilizar este recurso, se evitan lineas de codigo innecesarias, haciendo el proceso mas eficiente para **quien escribe el codigo** mas no para la **maquina**.

- Prompt para ChatGPT: ¿Cómo funciona el polimorfismo en C++ a nivel interno? Explica cómo se utilizan las vtables para resolver métodos virtuales en una jerarquía de herencia.  

    1. El programa encuentra la llamada a animal->makeSound()
    2. Como makeSound() es virtual, no llama directamente a la función
    3. En su lugar:

        - Accede al vptr del objeto (primeros bytes en memoria)
        - Usa el vptr para localizar la vtable asociada con la clase real del objeto
        - Busca en la vtable la dirección del método makeSound() implementado por esa clase
        - Salta a esa dirección y ejecuta la función correcta


        ```
        Objeto Dog:           Objeto Cat:  
        +--------+            +--------+  
        | vptr   |----------> | vptr   |--------+  
        +--------+    |       +--------+    |   |  
        | datos  |    |       | datos  |    |   |  
        +--------+    |       +--------+    |   |  
                    |                     |   |  
                    v                     v   |  
                vtable Dog:           vtable Cat:  
                +------------+        +------------+  
                | typeid     |        | typeid     |  
                +------------+        +------------+  
                | makeSound  |--+     | makeSound  |--+  
                +------------+  |     +------------+  |  
                                |                     |  
                                v                     v  
                        Dog::makeSound()     Cat::makeSound()  
                        (imprime "Bark")     (imprime "Meow")  
        ```

4. Reflexión Individual:

- ¿Cómo se implementan internamente el encapsulamiento, la herencia y el polimorfismo?  
El **encapsulamiento** permite limitar acceso libre a datos y protegerlos, esto mediante la declaracion de variables de la siguiente manera `(diferenciador) (tipo de dato)`:  
`public`: Permite que los datos sean modificados independientemente desde donde se haga.  
`protected`: Permite que los datos solo sean modificados desde su interior, o al interior de clases derivadas.  
`private`: Permite que los datos solamente sean accesibles desde el interior de la clase en la que fueron creados.  
A su vez, la **herencia** permite que se creen clases mayores, que derivan a clases mas especificas, de forma que escribir codigo no se convierta en algo redundante, y que no se creen atributos identicos a otros. Dicese de una clase madre, al momento de crear a la clase hijo se debe escribir de la siguiente manera: `class hijo : public madre`
Para finalizar, el **polimorfismo** se refiere a que distintos objetos de distintas clases respondan de forma distinta a una misma orden. Dicese de un metodo `makeSound()` escrito en una clase madre. Para permitir **polimorfismo** en sus clases derivadas, se agrega a estas un metodo de la siguiente manera: `void makeSound() override`.  

Ej:

```cpp
#include <iostream>

class Animal {
public: //encapsulamiento
    virtual void makeSound() {
        std::cout << "Some generic sound" << std::endl;
    }
};
class Dog : public Animal { //herencia
public: //encapsulamiento
    void makeSound() override { //polimorfismo
        std::cout << "Bark" << std::endl;
    }
};
class Cat : public Animal { //herencia
public: //encapsulamiento
    void makeSound() override { //polimorfismo
        std::cout << "Meow" << std::endl;
    }
};
```
- Análisis: ventajas y desventajas en términos de eficiencia y complejidad.  
En lo que se refiere a complejidad y eficiencia, utilizar este tipo de mecanismos que permiten incrementar la comprensibilidad y recortar la longitud del codigo hacen del proceso de escribirlo en algo mas comodo y eficiente para **el programador**, mas al momento de ejecutar el codigo, este tipo de mecanismos, especificamente el uso de **polimorfismos** hacen de la ejecucion en algo mas exigente para **la maquina**.