# RAE 1

### - La construccion de la aplicacion que propone el reto.
## ofApp.h

```cpp
#pragma once

#include "ofMain.h"
#include "of3dPrimitives.h"


class ofApp : public ofBaseApp {

public:
	void setup();
	void draw();
	float startTimer;
	float endTimer;

	
	ofShader shader;
	ofPlanePrimitive plane;
};
```

## ofApp.cpp

```cpp
#include "ofApp.h"

void ofApp::setup() {
    
    startTimer = ofGetElapsedTimef();
    endTimer = ofRandom(5, 10);
    float planeScale = 0.75;
    int planeWidth = ofGetWidth() * planeScale;
    int planeHeight = ofGetHeight() * planeScale;
    int planeGridSize = 20;
    int planeColumns = planeWidth / planeGridSize;
    int planeRows = planeHeight / planeGridSize;

    plane.set(planeWidth, planeHeight, planeColumns, planeRows, OF_PRIMITIVE_TRIANGLES);
    shader.load("shadersGL3/shader");;
}

void ofApp::draw() {
    float currentTime = ofGetElapsedTimef();
    shader.begin();
    float percentY = mouseY / (float)ofGetHeight(); //shader.vert
    percentY = ofClamp(percentY, 0, 1);
    shader.setUniform1f("percentY", percentY);
    float cx = ofGetWidth() / 2.0;
    float cy = ofGetHeight() / 2.0;

    float mx = mouseX - cx;
    float my = mouseY - cy;
    shader.setUniform1f("mouseRange", 150);
    shader.setUniform2f("mousePos", mx, my);

    ofTranslate(cx, cy);

    if (currentTime - startTimer >= endTimer) { //cambiador de color con temporizador.
        startTimer = currentTime;
        endTimer = 10 - (ofRandom(0, 500) / 100);
        shader.setUniform4f("color", ofRandom(1.0), ofRandom(1.0), ofRandom(1.0), 1.0);
    }

    plane.drawWireframe();
    shader.end();
}
```

## shader.vert

```cpp
#version 150


uniform mat4 modelViewProjectionMatrix;
in vec4 position;

uniform float percentY;
uniform vec2 mousePos;
uniform float mouseRange;

void main()
{
    
    vec4 pos = position;
    float dist =  distance(pos.xy, mousePos);

    if (percentY >= 0.5) { //si y esta mas alla de la mitad, repeler

        vec2 dir = pos.xy - mousePos;

        if (dist > 0.0 && dist < mouseRange) { 
            float distNorm = dist/mouseRange;
            distNorm = 1.0 - distNorm;
            dir *= distNorm;
            pos.x += dir.x;
            pos.y += dir.y;
        }

    }

    else if (percentY < 0.5) { //si y esta debajo de la mitad, atraer

        vec2 dir = pos.xy + mousePos;

        if (dist > 0.0 && dist < mouseRange) {
            float distNorm = dist/mouseRange;
            distNorm = 1.0 - distNorm;
            dir *= distNorm;
            pos.x += dir.x;
            pos.y += dir.y;

        }
    }
    
    gl_Position = modelViewProjectionMatrix * pos;
}
```

## Shader.frag

```cpp
// fragment shader

#version 150

uniform vec4 color;
out vec4 outputColor;

void main()
{
    outputColor = color;
}
```

### - Vas a explicar detalladamente como funciona la aplicacion.

Para empezar, en `ofApp.h` estan las clases habituales, sin embargo, en este programa se agregaron 2 atributos adicionales, el de `startTimer` y el de `endTimer`. Esto se hizo de esta manera debido a que debia poder inicializarlas desde el `void setup()` para que el programa pudiera operar normalmente.

Ahora, en `ofApp.cpp` empezamos con `void setup()`, desde el cual se inicializan tanto `startTimer`, `endTimer` y se crea el plano con los respectivos valores de sus atributos.  
Tras esto entramos a la carne del programa, `void draw()`, donde lo primero que se hace es crear un `float currentTime = ofGetElapsedTimef()`, este es crucial para un condicional que viene mas adelante. Tras esto se **inicializa el shader**. Desde aqui empezamos a hacer el set-up para que `shader.vert` funcione correctamente, empezando por obtener el valor de `percentY`, el cual es un decimal que representa la posicion del mouse en el rango **y** de la pantalla. Este valor procede a filtrarse mediante el uso de un `ofClamp`, que limitara sus valores a un rango **de 0 a 1**, para posteriormente enviarse al **Vertex Shader** a traves de un Uniform. Tras esto se procede a obtener los floats `cx` y `cy`, los cuales tienen un valor de la mitad del espacio que ocupara el aplicativo. Con estos valores `cx` y `cy` se hace el calculo para obtener otros 2 valores `mx` y `my`, esto debido a que se requiere escalar el movimiento del mouse de forma que este no se sienta antinatural en comparacion con el del plano, y acomoda el plano en la posicion que deberia estar, la mitad de la pantalla. Tras esto viene el codigo que aporta al **Fragment shader** de los atributos que requiere para funcionar. Empieza por un condicional `if (currentTime - startTimer >= endTimer)`, que es el que hace funcionar al temporizador en su totalidad, tras esto reinicia el temporizador, al actualizar `startTimer` con el valor de `currentTime`, de forma que la resta siga siendo funcional para el ciclo siguiente, y tras esto, actualiza el `endTimer` con un valor aleatorio entre 5 y 10 segundos. Finalmente, aun en el condicional, envia a `shader.frag` un nuevo valor para el color, que es el objetivo puntual de este codigo. Finalmente, se dibuja el **mesh** de `plane` y **se finalizan los procesos del shader**.  

En el archivo `shader.vert`, que es el primero en ejecutar sus funciones, se reciben los Uniforms `float percentY`, `vec2 mousePos` y `float mouseRange`, que seran utilizados en sus operaciones. Ahora, en el metodo que ejecuta este archivo, empezamos creando una copia del vector `position`, de forma que los valores de este puedan editarse libremente. Tras esto se crea un `float dist`, que cumple la funcion de **determinar cuanto debe alejarse o acercarse la particula** en caso de estar en dentro del `mouseRange`. Desde aqui se generan **2 condicionales mellizos**, el primero en caso de que `percentY >= 0.5`, en cuyo caso repelera los vertices mediante el `ver2 dir`, en cuya operacion se restara. El segundo mellizo se da en el caso `percentY < 0.5`, en cuyo caso atraera los vertices mediante el `ver2 dir`, en cuya operacion se sumara. EL resto del codigo al interior de los condicionales se encarga de que **la fuerza del movimiento funcione en torno a la cercania del vector al mouse** y de **agregar el movimiento hacia la direccion establecida** al vector de posicion que se puede editar. Tras esto, se le indica al shader que haga las operaciones de posicion respectivas **con el vector de posicion que se edito**.

Finalmente, en el archivo `shader.frag` se agrega como unico Uniform a `vec4 color`, el cual posteriormente se utiliza para determinar el `outputColor`.  

### Un ENLACE a un video que muestre en funcionamiento la aplicacion.

[Video evidencia](https://youtu.be/N8ARX0v6pJc)

# RAE2

- Explica y muestra cómo probaste la aplicación en ofApp.cpp.  
No tengo como evidenciar que lo hice, sencillamente estuve probando el programa hasta el fallo solucionando errores hasta que por fin se comportaba como necesitaba que lo hiciera.

- Explica y muestra cómo probaste el vertex shader.  
La forma en que puedo explicarlo es... Simplemente cumplia las funciones que debia de la forma en que debia, eso siempre fue claro desde la ejecucion del programa.  
- Explica y muestra cómo probaste el fragment shader.  
Pues, otra vez puedo explicarlo de una forma extraña. Cuando no funcionaba como debia, el plano se iba completamente a una esquina en la izquierda, ni siquiera su posicionamiento era correcto, fue hasta que dejo de hacerlo y que efectivamente cambiaba de colores a los que se comportaban de manera aleatoria que demostro que funcionaba.
- Explica y muestra cómo probaste toda la aplicación completa.  
Ejecutar, regresar, intentar resolver, repetir.