### Actividad 1

👍



### Actividad 2

Analicemos juntos este código:
¿Qué fue lo que incluimos en el archivo .h?  
R/ En este archivo se crea el objeto main con sus respectivos parámetros, siendo estos, públicamente, los voids requeridos para que el programa funcione, y las funciones para crear las partículas como un vector (unas seguidas de otras) y darles color.

¿Cómo funciona la aplicación?  
R/ Para empezar, en el void de `Setup` se establece el fondo de color negro y el color de las particulas, siendo este, blanco. Luego, en el void `Draw`, se indica la forma de las partículas (un círculo), y su color, siendo el mismo color que se establece en el Setup. Luego de esto, en el void `mouseMoved` se envia la informacion de estas partículas en el vector, es decir, baja en posición de prioridad; a su vez, en este mismo void, si el vector ya supera en datos incluidos, la cantidad de 100, se borra su última posición, eliminando la partícula correspondiente.  Para finalizar, en el void `mousePressed`, utilizando el parámetro particleColor, se randomiza este entre todos los colores admitidos por códigos RGB.

¿Qué hace la función mouseMoved?  
R/ La función mouseMoved detecta la posición del cursor en el espacio de la pantalla, y en caso de que este cambie respecto a en el frame anterior, ejecutando el código escrito dentro de esta.

¿Qué hace la función mousePressed?  
R/ La función mousePressed detecta si alguno de los botones del mouse, independiente del que sea, está siendo presionado, ejecutando el código escrito dentro de esta.

¿Qué hace la función setup?  
R/ La función Setup es una función que únicamente se cumple una única vez, que se ejecuta al inicio del programa.

¿Qué hace la función update?  
R/ La función update se encarga de ejecutarse en cada frame, en esta se realizan cálculos que necesite hacerse constantemente mientras el programa esté en uso.  

¿Qué hace la función draw?  
R/ La función draw se utiliza en, como dice su nombre, dibujar y mostrar en pantalla. En esta se ubica cualquier tipo de operación de dibujo, ya que es la única capaz de hacerla.|

### Actividad 3

```cpp


	#include "ofApp.h"

//--------------------------------------------------------------
void ofApp::setup(){
  	  ofBackground(0);
   	 particleColor = ofColor::white;
}

//--------------------------------------------------------------
void ofApp::update(){

}

//--------------------------------------------------------------
void ofApp::draw(){
    for(auto &pos: particles){
 	       ofSetColor(particleColor);
 	       ofDrawCircle(pos.x, pos.y, 50);
    }
}

//--------------------------------------------------------------
void ofApp::mouseMoved(int x, int y ){
    	particles.push_back(ofVec2f(x, y));
    	if (particles.size() > 100) {
    	    particles.erase(particles.begin());
    	}
}

//--------------------------------------------------------------
void ofApp::mousePressed(int x, int y, int button){
    	particleColor = ofColor(ofRandom(255), ofRandom(255), ofRandom(255));
}
	
```cpp
Ahora, el archivo modificado.
	#include "ofApp.h"

//--------------------------------------------------------------
void ofApp::setup(){
  	  ofBackground(0);
    particleColor = ofColor::white;
}
	
//--------------------------------------------------------------
void ofApp::update(){
	
}

//--------------------------------------------------------------
void ofApp::draw(){
    for(auto &pos: particles){
        ofSetColor(particleColor);
 	       ofDrawTriangle(pos.x + 30, pos.y + 30, pos.x - 30, pos.y + 30, pos.x, pos.y - 30);
  	  }
}

//--------------------------------------------------------------
void ofApp::mouseMoved(int x, int y ){
    particles.push_back(ofVec2f(x, y));
    if (particles.size() > 100) {
        particles.erase(particles.begin());
    }
}

//--------------------------------------------------------------
void ofApp::mousePressed(int x, int y, int button){
    particleColor = ofColor(ofRandom(255), ofRandom(255), ofRandom(255));
}
```
#### El circulo ahora es un triangulo.

### Actividad 4

👍


### Actividad 5

¿Cuál es la definición de un puntero?

R/ Un puntero es una casilla de RAM dedicada a almacenar la ubicación de otra casilla de RAM, donde se almacena algún dato que se quiera utilizar.

¿Dónde está el puntero?

R/ En este caso, el, o mas bien, **los** punteros se encuentran ubicados en las siguientes lineas de código:

```cpp
float getX(); 
float getY(); 
float getRadius();
```


¿Cómo se inicializa el puntero?

R/ Primero se establece el dato que quiere almacenar, en este caso, lo hace mediante generar la esfera, registrando para cada una de estas un valor `x`, un valor `y` y un radio. Tras esto, crea un float (quien tomará la labor de puntero) y le determina un valor de la siguiente manera:

```cpp
float getx()
```

Siguiendo este dado, se puede suponer entonces que al utilizar la función `get` seguido de la variable cuya ubicación se quiere almacenar, entonces esta seria la forma de almacenar un puntero en c++


¿Para qué se está usando el puntero?

Primero, dar una pequeña ojeada al código en el que se utiliza esta función.

```cpp
void ofApp::mousePressed(int x, int y, int button)
{ 
	if(button == OF_MOUSE_BUTTON_LEFT)
	{ 
		for (auto sphere : spheres) 
		{ 
			float distance = ofDist(x, y, sphere->getX(), sphere->getY()); 
			if (distance < sphere->getRadius()) 
				{ 
					selectedSphere = sphere; break; 
				} 
		} 
	} 
}
```

Okay, ahora que podemos ver que es lo que se está haciendo en este void, tener en consideración la línea `float distance = ofDist(x, y, sphere->getX(), sphere->getY());` y la línea `if (distance < sphere->getRadius())`, donde para poder recuperar la información de la esfera se utilizan las funciones de `get()`, de manera que se pueda obtener el dato dentro del puntero SIN modificarlo, dando una forma de protección y organización al programa, sin tener que generar funciones redundantes para cada esfera o tener que escribir líneas de código adicionales para detectar con cual se está interactuando.


¿Qué es exactamente lo que está almacenado en el puntero?

R/ En este caso, lo que está almacenado en el puntero, es una función que permite obtener el valor almacenado dentro de este. Por ejemplo, en el caso de `float getx()`, este hace que al momento de ejecutarse esta función, se obtenga el valor de x de la esfera con la que se está interactuando, siendo este uno de los parámetros de cada esfera creada, y permitiendo utilizarlo en los condicionales previamente mostrados.

### Actividad 6

El código, desde ofApp.h carece de un método para detectar en qué momento se libera el mouse. Teniendo esto en cuenta, en el archivo ofApp.h se agregaría este método a la clase ofBaseApp:

```cpp
class ofApp : public ofBaseApp
{ 
public: 
void setup(); 
void update(); 
void draw(); 
void mouseMoved(int x, int y ); 
void mousePressed(int x, int y, int button); 
Void mouseReleased(int x, int y, int button);

private: 
vector<Sphere*> spheres; 
Sphere* selectedSphere; 
};
```
Ya que se disponibiliza esta alternativa, pasando ahora al archivo ofApp.cpp, se agregaría el método correspondiente de la siguiente manera:

```cpp
void ofApp::mouseReleased(int x, int y, int button) 
{
    if(button == OF_MOUSE_BUTTON_LEFT) 
	{
        		selectedSphere = nullptr;
    	}
}

```

De esta manera, al detectar que el click izquierdo del mouse fue liberado, se liberaría también la esfera siendo seleccionada en el momento.


### Actividad 7

En el primer ejercicio, al momento de crear la esfera, esta se almacena dentro de la misma función. Al ser local, se almacena en la memoria automática. Tras finalizar la función, esta misma elimina la esfera, que ya había sido almacenada en el globalVector, que no depende de la función previamente mencionada, haciendo que su función como puntero apunte, valga la redundancia, a un dato que ya no existe, llevando al programa a colapsar tan pronto se le ordena dibujar la esfera. Debido a esto, cuando se presiona la tecla c, que ejecuta esta serie de acciones, el programa se congela y empieza a consumir la memoria en su totalidad, intentando obtener el dato almacenado al interior de globalVector, dato que ya no existe.

Cuando se hace el cambio del espacio de almacenamiento de la esfera al heap (memoria dinámica), esta deja de eliminarse cuando la función concluye, permitiendo al programa ejecutarse con normalidad. Gracias a esto, al presionar la tecla c, se dibuja y almacena la esfera creada.

### Actividad 8

ofApp.h

```cpp
#pragma once

#include "ofMain.h"

class Sphere {
public:
    Sphere(float x, float y, float radius);
    void draw() const;
    float x, y, radius;
    ofColor color;
};

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();
    void keyPressed(int key);

private:
   
    Sphere globalSphere{ 100, 100, 30 };

    
    std::vector<Sphere*> heapSpheres;

   
    void createStackSphere();
    Sphere* lastStackSphereRef = nullptr; 
};

```

ofApp.cpp

```cpp
#include "ofApp.h"

Sphere::Sphere(float x, float y, float radius) : x(x), y(y), radius(radius) {
    color = ofColor(ofRandom(255), ofRandom(255), ofRandom(255));
    ofLog() << "Sphere created at (" << x << ", " << y << ")";
}

void Sphere::draw() const {
    ofSetColor(color);
    ofDrawCircle(x, y, radius);
}

void ofApp::setup() {
    ofBackground(0);
    globalSphere.color = ofColor(255, 0, 0); 
}

void ofApp::update() {}

void ofApp::draw() {
    
    globalSphere.draw(); //dibujar esfera en global
    ofDrawBitmapString("Global Sphere (memoria global)", globalSphere.x, globalSphere.y - 40);

    
    for (Sphere* sphere : heapSpheres) {
        if (sphere) {
            sphere->draw();
            ofDrawBitmapString("Heap Sphere", sphere->x, sphere->y - 40);
        }
    }

    if (lastStackSphereRef) {
        ofSetColor(255, 255, 0); 
        ofDrawBitmapString("Stack Sphere (¿acceso inválido?)", lastStackSphereRef->x, lastStackSphereRef->y - 40);
        lastStackSphereRef->draw(); 
    }
}

void ofApp::keyPressed(int key) {
    if (key == 'h') { //dibujar esfera en el heap
        Sphere* newSphere = new Sphere(ofRandomWidth(), ofRandomHeight(), 30);
        heapSpheres.push_back(newSphere);
        ofLog() << "Heap Sphere creada (puntero: " << newSphere << ")";
    }
    else if (key == 's') { //dibujar esfera en stack
        createStackSphere();
    }
  }

void ofApp::createStackSphere() {
    Sphere stackSphere(ofRandomWidth(), ofRandomHeight(), 30); 
    lastStackSphereRef = &stackSphere; 
    ofLog() << "Stack Sphere creada (puntero: " << &stackSphere << ")";
}

```

¿Cuándo debo crear objetos en el heap y cuándo en memoria global?

R/ En la memoria global se generan objetos permanentes, es decir, aquellos que no pueden eliminarse. Ahora, al crear objetos en el heap, se espera que sean objetos de una duración variada, que no dependan del tiempo de duración de una función, pero que puedan eliminarse libremente para no abusar de los recursos de la máquina.

### Actividad 9

¿Qué sucede cuando presionas la tecla “f”?

R/ Cuando se presiona la tecla f se da un proceso similar a un equivalente del ctrl+z, es decir, desde que haya un paso hacia delante (considere esto como la generación de una esfera mediante la función ofApp::mousePressed()) este eliminará la primera esfera que haya en el array, es decir, la última que fue generada.
Analiza detalladamente esta parte del código:

```cpp
if(!heapObjects.empty()) 
{ 
	delete heapObjects.back(); 
	heapObjects.pop_back(); 
}
```

R/ Esta parte del código, es decir, la que elimina las esferas de la memoria heap, funciona de la siguiente manera: En primer lugar, pasa por un primer condicional, siendo este el void mismo, requiriendo que una tecla sea presionada para ejecutar el código dentro de esta. Tras esto, se enfrenta a un segundo condicional, siendo este si la tecla presionada fue la tecla “f”. Ahora, finalmente se ve el código mostrado. Este consta de un condicional primario, que se asegura de que en la lista de objetos almacenados en heap **NO** esté vacia. Tras esto, realiza 2 operaciones, la primera, llama al destructor de objetos, eliminando al que está marcado como “new”, es decir, al último que fue creado. Tras esto, para prevenir que se pueda dar un crash por dejar un puntero a un objeto que ya no existe, utiliza la línea `heapObjects.pop_back()`, la cual reduce el tamaño del vector de almacenamiento en uno, borrando así el puntero que podría causar que el programa crashee.


