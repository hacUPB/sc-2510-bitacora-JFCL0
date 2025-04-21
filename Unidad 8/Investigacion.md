# Actividad 1

### **1. Diferencia entre *programa* y *proceso***  
- **Programa**:  
  - Es un **archivo ejecutable** (por ej., `chrome.exe`, `main.py`) almacenado en disco.  
  - Contiene código binario, datos estáticos y metadatos (pero no se ejecuta por sí solo).  
- **Proceso**:  
  - Es una **instancia en ejecución** de un programa, cargado en memoria RAM.  
  - Tiene su propio espacio de direcciones, recursos (CPU, memoria, archivos abiertos) y estado (registros, stack, heap).  

**Ejemplo**:  
- El navegador Chrome es un *programa*.  
- Cada pestaña abierta es un *proceso* independiente.  

---

### **2. ¿Múltiples procesos ejecutando el mismo programa?**  
**¡Sí!** Puedes tener varios procesos corriendo el mismo programa. Cada uno tendrá:  
- Su propia memoria y recursos.  
- Estados independientes (ej., diferentes inputs o datos).  

**Ejemplo**:  
- Abrir 3 terminales ejecutando `/usr/bin/python3`: cada una es un proceso distinto.  

---

### **3. ¿Para qué sirve el *stack* de un proceso?**  
- Almacena:  
  - **Variables locales** de funciones.  
  - **Llamadas a funciones** (call stack): dirección de retorno, argumentos.  
  - Datos temporales (ej., resultados intermedios).  
- **Características**:  
  - Crece y se reduce automáticamente (LIFO: Last-In-First-Out).  
  - Tamaño fijo (si se llena, ocurre un *stack overflow*).  

---

### **4. ¿Para qué sirve el *heap* de un proceso?**  
- Almacena:  
  - **Memoria dinámica** (reservada con `malloc()`, `new`, etc.).  
  - Estructuras de datos grandes o de tamaño variable (ej., listas, objetos).  
- **Características**:  
  - Asignación/liberación manual (o con recolectores de basura).  
  - Fragmentación posible.  

**Ejemplo**:  
```c
int *arr = malloc(100 * sizeof(int)); // Se almacena en el heap.
```

---

### **5. ¿Qué es la *zona de texto* (text segment) de un proceso?**  
- Parte de la memoria que almacena:  
  - **Código máquina** del programa (instrucciones ejecutables).  
  - Es de **solo lectura** (para evitar modificaciones accidentales).  

**Ejemplo**:  
- Las funciones `main()`, `setup()`, `draw()` de tu programa están aquí.  

---

### **6. ¿Dónde se almacenan las variables globales *inicializadas*?**  
- En el **segmento de datos inicializados** (`data segment`).  
- Ejemplo:  
  ```c
  int x = 42;  // Variable global inicializada → data segment.
  ```

### **7. ¿Dónde se almacenan las variables globales *no inicializadas*?**  
- En el **segmento BSS** (Block Started by Symbol).  
- Se inicializan a `0`/`NULL` automáticamente.  
- Ejemplo:  
  ```c
  int y;  // Variable global no inicializada → BSS.
  ```

---

### **8. Estados posibles de un proceso**  
Los estados varían según el SO, pero en general:  
1. **Nuevo (New)**: Recién creado.  
2. **Listo (Ready)**: Esperando asignación de CPU.  
3. **En ejecución (Running)**: Usando la CPU.  
4. **Bloqueado/Espera (Waiting)**: Esperando un recurso (ej., I/O).  
5. **Terminado (Terminated)**: Finalizó su ejecución.  

**En sistemas Unix/Linux**:  
- Se añaden estados como:  
  - **Zombie**: Proceso terminado pero aún en la tabla de procesos.  
  - **Detenido (Stopped)**: Pausado por una señal (ej., `SIGSTOP`).  

**Diagrama típico**:  
```
New → Ready ↔ Running → Terminated  
          ↓      ↑  
          Waiting  
```

---

### **Resumen visual de la memoria de un proceso**  
```
+-------------------+  
|       Stack       | (Variables locales, LIFO)  
+-------------------+  
|        ↓          |  
|                   |  
|        ↑          |  
+-------------------+  
|       Heap        | (Memoria dinámica)  
+-------------------+  
|       BSS         | (Variables globales no init)  
+-------------------+  
|       Data        | (Variables globales init)  
+-------------------+  
|       Text        | (Código ejecutable)  
+-------------------+  
```

# Actividad 2

Vas a leer la sección del material de openframeworks, por ahora, solo la sección llamado: What’s a thread and when to use it. Luego de leer, Responde estas preguntas:

- ¿Qué es un hilo?  
Se refiere a un proceso cuya ejecucion **no depende del programa principal**, es decir, que puede ejecutarse en simultaneo del resto.
- ¿Cuándo se usa?  
Generalmente se usa en el caso de necesitar ejecutar una tarea que envuelva alguno de los componentes mas lentos del programa, como **la memoria**, o el **HD**.
- Trata de inferior esto: ¿A qué se puede referir el concepto de tarea ligada a la CPU y a la entrada/salida (E/S)?  
Se podria referir a: Las tareas ligadas a la CPU requieren que la maquina compiladora este la mayor parte del tiempo realizando operaciones, calculos matematicos, procesando algoritmos u ordenando arreglos. Las tereas ligadas a entrada/salida podrian referirse a tareas que hacen que la CPU este solicitando y recibiendo informacion de otros componentes constantemente.  
- ¿Cuál es la relación entre operaciones ligadas a la CPU y los hilos?  
Si hay muchos procesos conectados al uso de la CPU esto puede llevar a que algunos de estos no reciban el tiempo que requieren para ejecutarse, problema que puede solucionarse mediante el uso de hilos, de forma que multiples procesos mas pequeños se esten ejecutando simultaneamente.  
- ¿Cuál es la relación entre operaciones ligadas a la E/S y los hilos?  
En caso de ejecutarse una funcion bloqueante, la maquina tendra que quedarse esperando a que esta se complete para poder continuar, proceso que se puede evitar en su totalidad mediante el uso de hilos, generando un sistema de notificacion, de forma que tan pronto se finalice el proceso la CPU regrese de estar realizando otros procesos y continue en la parte del codigo siguiente.

# Actividad 3

Vas a explorar la necesidad de tener un hilo cuando un programa tiene que hacer una tarea intensiva ligada a la CPU.

Ejecuta el siguiente programa:

ofApp.h:

```cpp
#pragma once
#include "ofMain.h"

class ofApp : public ofBaseApp {
public:
    float x = 0;
    float speed = 3;
    float circleSize = 50;

    void setup();
    void draw();
    void mousePressed(int x, int y, int button);
    void heavyComputation();
};
```

ofApp.cpp:

```cpp
#include "ofApp.h"

void ofApp::setup() {
    ofSetFrameRate(60);
    ofSetWindowShape(400, 400);
}

void ofApp::draw() {
    ofBackground(220);
    ofSetColor(0);
    ofDrawCircle(x, ofGetHeight() / 2, circleSize);
    x = fmod(x + speed, ofGetWidth());
}

void ofApp::mousePressed(int x, int y, int button) {
    heavyComputation();
}

void ofApp::heavyComputation() {
    double result = 0;
    for (int j = 0; j < 1000000000; ++j) {
        result += sqrt(j);
    }
    circleSize = ofRandom(20, 70);
    ofLog() << "Circle size: " << circleSize;
}
```

- ¿Qué pasa cuando presionas el mouse?  
El programa se congela.
- ¿Por qué?  
La CPU se sobreexplota por la cantidad de 1) operaciones, 2) cosas que debe escribir en el log.
- Considerando la definición de un hilo, ¿cómo podrías solucionar el problema de que el programa se congele cuando presionas el mouse?  
ofApp.h
```cpp
#pragma once
#include "ofMain.h"

class HeavyCalc : public ofThread {

public:

    float circleSize = 50;

    void threadedFunction() {


        double result = 0;
        for (int j = 0; j < 1000000000; ++j) {
            result += sqrt(j);
        }
        circleSize = ofRandom(20, 70);
        ofLog() << "Circle size: " << circleSize;
    }
};

class ofApp : public ofBaseApp {
public:
    float x = 0;
    float speed = 3;
    bool loading;

    void setup();
    void draw();
    void mousePressed(int x, int y, int button);
    HeavyCalc heavyCalc;
};
```

# Actividad 4

Vas a solucionar el problema de la actividad anterior usando hilos.

Para solucionar el problema de la actividad anterior, vas a crear un hilo. Observa el siguiente código:

ofApp.h:

```cpp
#pragma once
#include "ofMain.h"
#include "ofThread.h"

class ofApp : public ofBaseApp, public ofThread {
public:
    float x = 0;
    float speed = 3;
    float circleSize = 50;

    void setup();
    void draw();
    void mousePressed(int x, int y, int button);
    void heavyComputation();
    void startHeavyComputation();
    void threadedFunction() override;
    void exit();
};
```

ofApp.cpp:

```cpp
#include "ofApp.h"

void ofApp::setup() {
    ofSetFrameRate(60);
    ofSetWindowShape(400, 400);
}

void ofApp::draw() {

    ofBackground(220);
    ofSetColor(0);
    lock();
    ofDrawCircle(x, ofGetHeight() / 2, circleSize);
    unlock();
    x = fmod(x + speed, ofGetWidth());
}

void ofApp::mousePressed(int x, int y, int button) {
    if (!isThreadRunning()) {
        ofLog() << "Starting thread";
        startThread();
    }
    else {
        ofLog() << "Thread is already running";
    }
}

void ofApp::threadedFunction() {
    heavyComputation();
    ofLog() << "Thread ends";
}

void ofApp::heavyComputation() {
    double result = 0;
    for (int j = 0; j < 1000000000; ++j) {
        result += sqrt(j);
    }
    lock();
    ofSeedRandom();
    circleSize = ofRandom(20, 70);
    unlock();
    ofLog() << "Circle size: " << circleSize;
}

void ofApp::exit() {
    if (isThreadRunning()) {
        stopThread();
        waitForThread();
    }
}
```

- ¿Qué pasa cuando presionas el mouse?  
Genera 1 condicional que analiza si hay un Thread ejecutandose, en caso de que no inicia un thread y lo registra en la consola, en caso de que si registra en la consola que ya hay un thread en ejecucion. El thread que ejecuta permite ejecutar el metodo `heavyComputation` en simultaneo de que el aplicativo sigue ejecutandose.
- ¿Por qué?  
El condicional existe para evitar colapsar el sistema generando una infinidad de threads complejos, limitandolo a solamente 1. `heavyComputation` primero genera una tarea pesada para dar un motivo al programa de utilizar un Thread para manejarlo, y tras realizarla, randomiza el tamaño del circulo que se dibuja en el void `draw()`.
- ¿Qué hace el hilo?  
El hilo ejecuta la funcion `heavyComputation` y tras completar esa funcion envia una notificacion de completacion por consola.
- ¿Por qué el programa no se congela cuando presionas el mouse?  
Ya que la tarea de `heavyComputation` ya no se procesa al interior del thread `main` del programa, sino que se ejecuta en el thread agregado.
- ¿Qué pasa si presionas el mouse varias veces?  
El programa revisa el condicional, y segun este, respondera denegando la capacidad de agregar mas threads si ya hay uno en proceso.
- ¿Por qué?  
Para no abusar de los recursos de la CPU y, en consecuencia, de arriesgar un posible crash.
- ¿Qué pasa si presionas el mouse varias veces y rápido?  
Simplemente tirara el mismo mensaje de que ya hay un thread en progreso si es asi.
- ¿Por qué?  
Otra vez, no abusar de la capacidad de la CPU.
- Para qué se están usando las funciones lock y unlock?  
Para prevenir el acceso del thread principal a memoria, esto lo hace para evitar que mutiples threads esten intentando acceder a la memoria en simultaneo. En este caso lo que haria es, durante el breve momento en que esta bloqueado, cambiar el valor de `ofSeedRandom()` y `circleSize`, ambas variables, y bloquea el acceso a este para evitar que `draw()` se ejecute por tan solo un momento, y tan pronto se hace el cambio se vuelve a desbloquear. El `void draw()` tiene esta misma funcionalidad, de forma que al momento de interactuar con las variables del circulo, esto no se haga al mismo tiempo.
- ¿Qué está pasando en la función exit?  
Ejecuta 2 funciones de la clase `ofThread`, siendo estas co-dependientes. Primero se da la orden de detener el proceso mediante la linea de codigo `stopThread`, que desde la clase `ofThread` da la orden de establecer el `ThreadRunning` a un valor **false**. Su ultima linea de codigo `waitForThread()` le indica al programa que verifique que el proceso, efectivamente, se detuvo (este metodo es preferible para funciones que contienen un loop-while).

# Actividad 5

Ahora te propondré un caso de estudio en el cual vas a comparar un programa con hilos y otro sin hilos resolviendo el mismo problema: flocking.

Primero te mostraré la versión sin hilos:

ofApp.h:

```cpp
#pragma once

#include "ofMain.h"

class Boid {
public:
    Boid(float x, float y);
    void run(const vector<Boid>& boids);
    void applyForce(ofVec2f force);
    void flock(const vector<Boid>& boids);
    void update();
    void borders();
    ofVec2f seek(ofVec2f target);
    void draw();

    ofVec2f separate(const vector<Boid>& boids);
    ofVec2f align(const vector<Boid>& boids);
    ofVec2f cohere(const vector<Boid>& boids);

    ofVec2f position;
    ofVec2f velocity;
    ofVec2f acceleration;
    float r;
    float maxforce;
    float maxspeed;
};


class Flock {
public:
    void addBoid(float x, float y);
    void run();

    vector<Boid> boids;
};


class ofApp : public ofBaseApp {

public:
    void setup();
    void update();
    void draw();
    void mouseDragged(int x, int y, int button);

    Flock flock;
};
```

ofApp.cpp (comentado)

```cpp
#include "ofApp.h"

Boid::Boid(float x, float y) { // inicializar Boid
    acceleration.set(0, 0);
    velocity.set(ofRandom(-1, 1), ofRandom(-1, 1));
    position.set(x, y);
    r = 3.0;
    maxspeed = 3;
    maxforce = 0.05;
}

void Boid::run(const vector<Boid>& boids) {
    flock(boids); // ejecutar void flock de la clase boid
    update(); // ejecutar void update de la clase boid
    borders(); // ejecutar void borders de la clase boid
}

void Boid::applyForce(ofVec2f force) {
    acceleration += force; // modificar el valor del atributo aceleracion agregandole el valor recibido de la fuerza
}

void Boid::flock(const vector<Boid>& boids) {
    ofVec2f sep = separate(boids); // ejecutar metodo separate
    ofVec2f ali = align(boids); // ejecutar metodo align
    ofVec2f coh = cohere(boids); // ejecutar metodo cohere
    sep *= 1.5; // modificar valor obtenido en separate
    ali *= 1.0; // modificar valor obtenido en align
    coh *= 1.0; // modificar valor obtenido en cohere

    applyForce(sep); // ejecutar void applyForce de la clase Boid
    applyForce(ali); // ejecutar void applyForce de la clase Boid
    applyForce(coh); // ejecutar void applyForce de la clase Boid
}

void Boid::update() {
    velocity += acceleration; // incrementar el atributo de velocidad mediante agregarle el atributo de aceleracion
    velocity.limit(maxspeed); // aplicar el limite de maximo de velocidad, de forma que si la linea anterior se paso del valor este se convierta en el valor de maxspeed
    position += velocity; // modificar la posicion del boid al agregarle su velocidad
    acceleration *= 0; // igualar aceleracion a 0
}

ofVec2f Boid::seek(ofVec2f target) {
    ofVec2f desired = target - position; // crea un nuevo vector, resultado de sustraer la posicion del boid al promedio de las posiciones de todos los vectores que tiene dentro del rango para considerarles vecinos
    desired.normalize(); // convierte el nuevo vector en un rango de 0 a 1
    desired *= maxspeed; // multiplica el nuevo vector por la velocidad maxima que puede alcanzar el boid
    ofVec2f steer = desired - velocity; // crea un nuevo vector steer, cuyo valor se obtiene restando
    steer.limit(maxforce); // le establece el limite de fuerza al vector que creo, con el fin de limitar sus rangos
    return steer; // envia el vector steer de vuelta al void flock de boid
}

void Boid::borders() {

    float width = ofGetWidth(); // obtener grosor del canvas y almacenarlo en este float
    float height = ofGetHeight(); // obtener altura del canvas y almacenarla en este float

    // los boids se construyen de manera que en Model Space la posicion de sus pixeles se modifica unicamente en torno a r.
    // Al momento de considerar las posiciones se imagina un rango aumentado al tamaño de la pantalla, de forma que tan pronto el boid deja de ser visible este es transportado al lado opuesto de la pantalla para generar un efecto de pantalla infinita.
    if (position.x < -r) position.x = width + r; // Si position.x se sobrepasa por -r de x = 0, su posicion en x se transforma en el ancho de la pantalla + r.
    if (position.y < -r) position.y = height + r; // Si position.y se sobrepasa por -r de y = 0, su posicion en y se transforma en la altura de la pantalla + r.
    if (position.x > width + r) position.x = -r; // Si position.x se sobrepasa por r del ancho de la pantalla, su posicion en x se transforma en x = 0 - r.
    if (position.y > height + r) position.y = -r; // Si position.y se sobrepasa por r de la altura de la pantalla, su posicion en y se transforma en y = 0 - r.

}

void Boid::draw() {
    float theta = atan2(velocity.y, velocity.x) + PI / 2; // obtiene el angulo generado por el vector velocidad en torno al eje x.
    ofSetColor(255); // se le asigna el color blanco.
    ofFill(); // le indica al renderer que llene la figura
    ofPushMatrix(); // empuja la matriz actual
    ofTranslate(position.x, position.y); // envia la particula a los valores almacenados en su atributo ofVec3f position
    ofRotate(ofRadToDeg(theta)); // rota la particula los grados obtenidos a partir de su velocidad y almacenados en theta
    ofBeginShape(); // inicializa la creacion de una forma a partir de vectores enlistados inmediatamente despues
    ofVertex(0, -r * 2); // establece la posicion del primer vector en Model Space
    ofVertex(-r, r * 2); // establece la posicion del segundo vector en Model Space
    ofVertex(r, r * 2); // establece la posicion del tercer vector en Model Space
    ofEndShape(); // finaliza la creacion de una forma a partir de vectores enlistados inmediatamente antes
    ofPopMatrix(); //trae de vuelta la posicion de las cordenadas que habia empujado antes
}

ofVec2f Boid::separate(const vector<Boid>& boids) {
    float desiredSeparation = 25; // inicializa float de separacion deseada entre particulas
    ofVec2f steer(0, 0); // inicializa vector steer
    int count = 0; // inicializa contador

    for (const Boid& other : boids) { // por cada boid diferente a el que se esta calculando en el vector boids
        float d = position.distance(other.position); // calcula la distancia entre su posicion y la del otro vector
        if (d > 0 && d < desiredSeparation) { // si d es mayor a 0 y menor a la separacion deseada ejecuta el codigo
            ofVec2f diff = position - other.position; // calcula la direccion desde el otro vector hacia este
            diff.normalize(); // hace que la direccion sean valores de 0 a 1
            diff /= d; // hace que a mas lejos este mas fuerte sea el efecto de diff
            steer += diff; // modifica el valor de steer por el valor calculado de diff
            count++; // agrega 1 al contador

        }
    }
    if (count > 0) { // si el contador es mayor a 0 (se ejecuto el bucle for)
        steer /= (float)count; // se saca un promedio de todos los valores de diff que se obtuvieron previamente.
    }

    if (steer.length() > 0) { // si steer tiene un valor almacenado mayor a 0 (se ejecutaron las 2 funciones anteriores)
        steer.normalize(); // hacer que steer sea un valor entre 0 y 1
        steer *= maxspeed; // multiplicar steer por el valor de velocidad maximo
        steer -= velocity; // restar al nuevo valor de steer la velocidad
        steer.limit(maxforce); // reconocer a steer como una fuerza y ponerle el limite que tienen todas las otras fuerzas
    }
    return steer; // devolver valor de separate al void Flock
}

ofVec2f Boid::align(const vector<Boid>& boids) {
    float neighborDist = 50; //establecer distancia esperada con los boids vecinos
    ofVec2f sum(0, 0); // generar vector sum e inicializarlo en 0
    int count = 0; // inicializar conteo

    for (const Boid& other : boids) { // por cada boid almacenado en el vector boids, consideremoslo como boid2
        float d = position.distance(other.position); // calcula la distancia entre su posicion y la del otro vector
        if (d > 0 && d < neighborDist) { // si la distancia entre el boid1 y el boid2 es mayor que 0 y menor que la distancia que deberia tener con su vecino
            sum += other.velocity; // agregar a sum la velocidad del boid2
            count++; // incrementar conteo
        }
    }
    if (count > 0) { // si el contador es mayor a 0 (se ejecuto el bucle for)
        sum /= (float)count; // sacar el promedio de velocidad de todos los boids diferentes al que se esta monitoreando
        sum.normalize(); // hacer que sum tenga un valor entre 0 y 1
        sum *= maxspeed; // multiplicar a sum la velocidad maxima
        ofVec2f steer = sum - velocity; // generar vector steer, siendo este igual a sum - la velocidad del vector en ese momento
        steer.limit(maxforce); // dar a steer el modificador de limite de fuerza, con el fin de poder enviarlo
        return steer; // retornar valor de align al void Flock
    }
    return ofVec2f(0, 0); // en caso de fallar, enviar un valor de align = 0.
}

ofVec2f Boid::cohere(const vector<Boid>& boids) {
    float neighborDist = 50; // establecer distancia esperada con los boids vecinos
    ofVec2f sum(0, 0); // generar vector sum e inicializarlo en 0
    int count = 0; // inicializar contador

    for (const Boid& other : boids) { // por cada boid almacenado en el vector boids
        float d = position.distance(other.position); // generar distancia entre el boid siendo monitoreado y el boid vecino de este turno
        if (d > 0 && d < neighborDist) { // si la distancia es mayor a 0 y menor que la distancia esperada 
            sum += other.position; // agregar al vector sum el valor de la posicion del boid vecino
            count++; // incrementar contador
        }
    }
    if (count > 0) { // si el contador es mayor a 0 (el loop anterior si se ejecuto)
        sum /= count; // sacar valor promedio de todos los valores sumados de sum
        return seek(sum); // retornar valor del cohere en al void seek
    }
    return ofVec2f(0, 0); // en caso de fallar o intentar ejecutar sin ningun boid en el vector
}

void Flock::addBoid(float x, float y) {
    boids.emplace_back(x, y); // insertar un boid en el vector boids
}

void Flock::run() {
    for (Boid& b : boids) {
        b.run(boids); // ejecutar void run de la clase boid por cada objeto b en el vector boids
    }
}


void ofApp::setup() {
    flock.boids.reserve(120); // asegura que los vectores de flock que contienen boids puedan contener como minimo 120 elementos.
    for (int i = 0; i < 120; i++) {
        flock.addBoid(ofGetWidth() / 2, ofGetHeight() / 2); //agrega un boid al vector de estos en el objeto flock un total de 120 veces (loop-for)
    }
}

void ofApp::update() {
    flock.run(); //ejecutar void run de la clase flock
}

void ofApp::draw() {
    ofBackground(0); // hacer que el fondo sea de color negro
    for (Boid& b : flock.boids) { // por cada boid en el vector boids
        b.draw(); // ejecutar metodo dibujar de la clase boid
    }
    ofDrawBitmapStringHighlight("FPS: " + ofToString(ofGetFrameRate()), 20, 20); // texto
    ofDrawBitmapStringHighlight("Boids: " + ofToString(flock.boids.size()), 20, 40); // texto

}

void ofApp::mouseDragged(int x, int y, int button) {
    flock.addBoid(x, y); // crea nuevos boids si se sostiene click y se mueve el mouse
}
```

Ahora mira la version con hilos:

ofApp.h

```cpp
#pragma once

#include "ofMain.h"
#include "ofThread.h"

class Boid {
public:
    Boid(float x, float y);
    void run(const vector<Boid>& boids);
    void applyForce(ofVec2f force);
    void flock(const vector<Boid>& boids);
    void update();
    void borders();
    ofVec2f seek(ofVec2f target);
    void draw();

    ofVec2f separate(const vector<Boid>& boids);
    ofVec2f align(const vector<Boid>& boids);
    ofVec2f cohere(const vector<Boid>& boids);

    ofVec2f position;
    ofVec2f velocity;
    ofVec2f acceleration;
    float r;
    float maxforce;
    float maxspeed;
};


class Flock : public ofThread { // Cambio: Ahora hereda de la clase ofThread
public:
    void addBoid(int x, int y); // Cambio: Paso de recibir floats a ints para x y y
    void run();
    void threadedFunction(); // Cambio: Nuevo metodo

    std::vector<Boid> boids;
};


class ofApp : public ofBaseApp {

public:
    void setup();
    void update();
    void draw();
    void exit(); // Cambio: Nuevo metodo
    void mouseDragged(int x, int y, int button);

    Flock flock;
};
```

ofApp.cpp
```cpp
#include "ofApp.h"

Boid::Boid(float x, float y) {
    acceleration.set(0, 0);
    velocity.set(ofRandom(-1, 1), ofRandom(-1, 1));
    position.set(x, y);
    r = 3.0;
    maxspeed = 3;
    maxforce = 0.05;
}

void Boid::run(const vector<Boid>& boids) {
    flock(boids);
    update();
    borders();
}

void Boid::applyForce(ofVec2f force) {
    acceleration += force;
}

void Boid::flock(const vector<Boid>& boids) {
    ofVec2f sep = separate(boids);
    ofVec2f ali = align(boids);
    ofVec2f coh = cohere(boids);
    sep *= 1.5;
    ali *= 1.0;
    coh *= 1.0;

    applyForce(sep);
    applyForce(ali);
    applyForce(coh);
}

void Boid::update() {
    velocity += acceleration;
    velocity.limit(maxspeed);
    position += velocity;
    acceleration *= 0;
}

ofVec2f Boid::seek(ofVec2f target) {
    ofVec2f desired = target - position;
    desired.normalize();
    desired *= maxspeed;
    ofVec2f steer = desired - velocity;
    steer.limit(maxforce);
    return steer;
}

void Boid::borders() { 
    // Cambio: remueve lineas para obtener width y height, poniendo las funciones ofGet en su lugar.
    if (position.x < -r) position.x = ofGetWidth() + r;
    if (position.y < -r) position.y = ofGetHeight() + r;
    if (position.x > ofGetWidth() + r) position.x = -r;
    if (position.y > ofGetHeight() + r) position.y = -r;
}

void Boid::draw() {
    float theta = atan2(velocity.y, velocity.x) + PI / 2;
    ofSetColor(255);
    ofFill();
    ofPushMatrix();
    ofTranslate(position.x, position.y);
    ofRotate(ofRadToDeg(theta));
    ofBeginShape();
    ofVertex(0, -r * 2);
    ofVertex(-r, r * 2);
    ofVertex(r, r * 2);
    ofEndShape();
    ofPopMatrix();
}

ofVec2f Boid::separate(const vector<Boid>& boids) {
    float desiredSeparation = 25;
    ofVec2f steer(0, 0);
    int count = 0;

    for (const Boid& other : boids) {
        float d = position.distance(other.position);
        if (d > 0 && d < desiredSeparation) {
            ofVec2f diff = position - other.position;
            diff.normalize();
            diff /= d;
            steer += diff;
            count++;
        }
    }
    if (count > 0) {
        steer /= (float)count;
    }

    if (steer.length() > 0) {
        steer.normalize();
        steer *= maxspeed;
        steer -= velocity;
        steer.limit(maxforce);
    }
    return steer;
}

ofVec2f Boid::align(const vector<Boid>& boids) {
    float neighborDist = 50;
    ofVec2f sum(0, 0);
    int count = 0;

    for (const Boid& other : boids) {
        float d = position.distance(other.position);
        if (d > 0 && d < neighborDist) {
            sum += other.velocity;
            count++;
        }
    }
    if (count > 0) {
        sum /= (float)count;
        sum.normalize();
        sum *= maxspeed;
        ofVec2f steer = sum - velocity;
        steer.limit(maxforce);
        return steer;
    }
    return ofVec2f(0, 0);
}

ofVec2f Boid::cohere(const vector<Boid>& boids) {
    float neighborDist = 50;
    ofVec2f sum(0, 0);
    int count = 0;

    for (const Boid& other : boids) {
        float d = position.distance(other.position);
        if (d > 0 && d < neighborDist) {
            sum += other.position;
            count++;
        }
    }
    if (count > 0) {
        sum /= count;
        return seek(sum);
    }
    return ofVec2f(0, 0);
}

void Flock::addBoid(int x, int y) { // Cambio: Ahora recibe int en lugar de floats.
    lock(); // Cambio: Agregar lock()
    boids.emplace_back(x,y);
    unlock(); // Cambio: Agregar unlock()
}

void Flock::run() {
    // Cambio: Remueve el bucle for(Boid& b : boids) y sus respectivos contenidos.
    // Cambio: Agrega el condicional if(!isThreadRunning) y sus respectivos contenidos.
    if (!isThreadRunning()) {
        startThread();
    }
}

void Flock::threadedFunction() { // Cambio: es todo completamente nuevo.
    while (isThreadRunning()) { 
        lock(); // bloquea acceso a atributos por parte de otros metodos.
        for (Boid& b : boids) {
            b.run(boids); // ejecuta void Boids::run
        }
        unlock(); // regresa libre acceso a atributos por parte de otros metodos
        sleep(10); // descansa por un momento
    }
}

void ofApp::setup() {
    // Cambio: Remueve la reserva de flock.boids.
    for (int i = 0; i < 120; i++) {
        flock.addBoid(ofGetWidth() / 2, ofGetHeight() / 2);
    }
}

void ofApp::update() {
    flock.run();
}

void ofApp::draw() {
    ofBackground(0);

    flock.lock(); // Cambio: Agrega bloqueo.
    for (Boid& b : flock.boids) {
        b.draw();
    }
    flock.unlock(); // Cambio: Agrega desbloqueo.

    ofDrawBitmapStringHighlight("FPS: " + ofToString(ofGetFrameRate()), 20, 20);
    ofDrawBitmapStringHighlight("Boids: " + ofToString(flock.boids.size()), 20, 40);
}

void ofApp::mouseDragged(int x, int y, int button) {
    flock.addBoid(x,y);
}

void ofApp::exit() { // Cambio: Esto es completamente nuevo.
    flock.stopThread();
    flock.waitForThread();
}
```

- **Estudia en detalle el código de ambas versiones. ¿Cómo funciona cada versión del programa?**
Revisar comentarios en los codigos.

- **¿Cómo se incorpora el concepto de hilo en la versión con hilos?**  
Se implementa mediante generar una herencia a la clase **Flock** de la siguiente manera: `class Flock : public ofThreads()`. Esta herencia afecta a los metodos en los que la clase Flock es llamada, ya que al momento de acceder a los datos, estos se **bloquean**, y tan pronto finaliza su tarea, esta los **desbloquea**. A su vez, se da un cambio sumamente significativo al metodo `Flock::run()`, el cual en lugar de ordenar al programa a ejecutar el metodo `Boids::run()` y continuar el constante loop de ejecutar **draw()->update()->draw()** ahora tiene integrado un condicional que, de no cumplirse, permitiria al programa seguir su curso de forma normal, y a su vez, **ejecutar estos mismos metodos run() en simultaneo con otras operaciones**. A su vez otro cambio significativo es la creacion de 2 nuevos metodos `Flock::threadedFunction()`, el cual se ejecutara en el fondo, siendo este metodo el que le ordena a los **boids** que deben **actualizar sus comportamientos**, no haciendo de estos arduos procesos en algo **tan exigente en tiempos de carga y ejecucion**. Y finalmente, el ultimo metodo que crea es el metodo `ofApp::exit()`, el cual detiene los procesos del **Thread** tan pronto se finaliza la ejecucion del aplicativo.

- **¿Qué es lock y unlock? ¿Por qué es importante realizar esta sincronización?** ¿Qué pasaría si no se realiza?  
No utilizar estos metodos en el codigo puede resultar en **conflictos al momento de acceder y modificar datos**, generando posibles **crashes** en el programa.

- **Compara la eficiencia de ambas versiones. ¿Cuál es más eficiente? ¿Por qué?**  
Haciendo un experimento sencillo, basado en crear la mayor cantidad posible de objetos **Boid**, resultando en un numero de estos objetos, cuya existencia en simultaneo empezaba a afectar el rendimiento del programa. El primer programa alcanzo un aproximado de 500 objetos antes de empezar a mostrar serias fallas de rendimiento, mas el codigo modificado logro alcanzar un aproximado de 750 antes de sufrir el mismo efecto.
- **El uso de lock y unlock en la versión con hilos, ¿Cómo afecta la eficiencia del programa?**  
Estos metodos afectan la eficiencia del programa ya que generan colas de espera para que otros Threads obtengan acceso a datos viniendo de espacios externos a la CPU, sin embargo, la diferencia es menor en comparacion con los beneficios que ofrece.