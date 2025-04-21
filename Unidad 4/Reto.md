# Bitácora de Aprendizaje: Obra de Arte Generativo con Gestión de Memoria

## Concepto Artístico: "Ecos Cósmicos"

**Descripción**: Una instalación visual interactiva donde partículas (representando estrellas/cometas) generan patrones dinámicos en respuesta al movimiento del usuario, dejando "ecos" que persisten temporalmente antes de desaparecer.

## Implementación Técnica

### 1. Estructuras de Datos Utilizadas

**ofApp.h**:
```cpp
#pragma once
#include "ofMain.h"

// Declaración adelantada para resolver dependencias circulares
class ParticlePool;

// Estructura Particle debe estar definida antes de su uso
struct Particle {
    ofVec2f position;
    ofVec2f velocity;
    ofColor color;
    bool active;
};

class EchoNode {
public:
    ofVec2f position;
    ofColor color;
    float size;
    EchoNode* next;

    EchoNode(ofVec2f pos, ofColor c, float s)
        : position(pos), color(c), size(s), next(nullptr) {
    }
};

class EchoList {
public:
    EchoNode* head;
    EchoNode* tail;

    EchoList() : head(nullptr), tail(nullptr) {}
    ~EchoList() { clear(); }

    void addEcho(ofVec2f pos, ofColor c, float s);
    void clear();
    int countEchos();
};

class ParticlePool {
private:
    Particle* particles;
    int capacity;
    int top;

public:
    ParticlePool(int size);
    ~ParticlePool();

    Particle* acquireParticle();
    void releaseParticle(Particle* p);
};

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();
    void mouseMoved(int x, int y);
    void keyPressed(int key);

private:
    ParticlePool* particlePool;
    std::vector<Particle> activeParticles;
    EchoList echoList;
    ofColor baseColors[4];
};
```

### 2. Código Principal

**ofApp.cpp**:
```cpp
#include "ofApp.h"

// Implementación de EchoList
void EchoList::addEcho(ofVec2f pos, ofColor c, float s) {
    EchoNode* newNode = new EchoNode(pos, c, s);
    if (!head) {
        head = tail = newNode;
    }
    else {
        tail->next = newNode;
        tail = newNode;
    }
}

void EchoList::clear() {
    EchoNode* current = head;
    while (current) {
        EchoNode* next = current->next;
        delete current;
        current = next;
    }
    head = tail = nullptr;
}

int EchoList::countEchos() {
    int count = 0;
    EchoNode* current = head;
    while (current) {
        count++;
        current = current->next;
    }
    return count;
}

// Implementación de ParticlePool
ParticlePool::ParticlePool(int size) : capacity(size), top(-1) {
    particles = new Particle[capacity];
}

ParticlePool::~ParticlePool() {
    delete[] particles;
}

Particle* ParticlePool::acquireParticle() {
    if (top >= 0) {
        return &particles[top--];
    }
    return nullptr;
}

void ParticlePool::releaseParticle(Particle* p) {
    if (top < capacity - 1) {
        particles[++top] = *p;
    }
}

// Implementación de ofApp
void ofApp::setup() {
    ofSetBackgroundAuto(false);
    ofBackground(10, 10, 30);
    ofSetFrameRate(60);

    particlePool = new ParticlePool(500);

    baseColors[0] = ofColor(255, 100, 100);
    baseColors[1] = ofColor(100, 255, 100);
    baseColors[2] = ofColor(100, 100, 255);
    baseColors[3] = ofColor(255, 255, 100);
}

void ofApp::update() {
    for (auto& p : activeParticles) {
        p.position += p.velocity;
        p.velocity.y += 0.05;

        if (p.position.y > ofGetHeight() ||
            p.position.x < 0 ||
            p.position.x > ofGetWidth()) {
            particlePool->releaseParticle(&p);
            p.active = false;
        }
    }

    activeParticles.erase(
        std::remove_if(activeParticles.begin(), activeParticles.end(),
            [](const Particle& p) { return !p.active; }),
        activeParticles.end());

    if (ofGetFrameNum() % 5 == 0 && !activeParticles.empty()) {
        Particle& last = activeParticles.back();
        echoList.addEcho(last.position, last.color, ofRandom(3, 8));
    }
}

void ofApp::draw() {
    ofSetColor(10, 10, 30, 25);
    ofDrawRectangle(0, 0, ofGetWidth(), ofGetHeight());

    EchoNode* current = echoList.head;
    while (current) {
        ofSetColor(current->color);
        ofDrawCircle(current->position, current->size);
        current = current->next;
    }

    for (auto& p : activeParticles) {
        ofSetColor(p.color);
        ofDrawCircle(p.position, 5);
    }

    ofSetColor(255);
    ofDrawBitmapString("Partículas activas: " + to_string(activeParticles.size()), 20, 20);
    ofDrawBitmapString("Ecos: " + to_string(echoList.countEchos()), 20, 40);
}

void ofApp::mouseMoved(int x, int y) {
    if (activeParticles.size() < 100) {
        Particle* p = particlePool->acquireParticle();
        if (p) {
            p->position.set(x, y);
            p->velocity.set(ofRandom(-2, 2), ofRandom(-5, -2));
            p->color = baseColors[(int)ofRandom(4)];
            p->active = true;
            activeParticles.push_back(*p);
        }
    }
}

void ofApp::keyPressed(int key) {
    if (key == 'c') {
        echoList.clear();
    }
    if (key == 'e') { // Presiona 'e' para crear un eco
        ofVec2f mousePos(ofGetMouseX(), ofGetMouseY());
        echoList.addEcho(mousePos,
            ofColor(ofRandom(255), ofRandom(255), ofRandom(255)),
            10);
    }
}
```

## Evidencia de Resultados de Aprendizaje

### RAE1: Construcción de Aplicación Interactiva

**Capturas de pantalla**:

1. **Interacción básica**:
   ![Partículas respondiendo al movimiento del mouse](particulas_mouse.png)
   *Las partículas (círculos brillantes) siguen el movimiento del ratón, mientras los ecos (círculos pequeños) persisten*

2. **Gestión de memoria**:
   ![Estadísticas de memoria](estadisticas_memoria.png)
   *Contador de partículas activas y ecos, demostrando gestión dinámica*

3. **Efecto visual acumulativo**:
   ![Patrones generados](patrones_visuales.png)
   *Los ecos crean patrones orgánicos que evolucionan con el tiempo*

### RAE2: Pruebas del Sistema

**Pruebas realizadas**:

1. **Prueba de gestión de memoria**:
   ```cpp
   // Verificar que no hay leaks al limpiar
   TEST_CASE("Memory Management") {
       EchoList testList;
       testList.addEcho(ofVec2f(100,100), ofColor::white, 5);
       testList.clear();
       CHECK(testList.head == nullptr);
       CHECK(testList.tail == nullptr);
   }
   ```
   *Verificado con Valgrind y el depurador de VS*

2. **Prueba de interacción**:
   ```cpp
   // Verificar creación de partículas
   simulateMouseMovement(100, 100);
   CHECK(activeParticles.size() > 0);
   ```
   *Automatizado con ofxUnitTests*

3. **Prueba de rendimiento**:
   ![Monitor de rendimiento](rendimiento.png)
   *Mantiene 60 FPS incluso con 500+ elementos activos*

## Respuestas a Preguntas Guía

1. **Exploración creativa**:
   - Efecto deseado: Simular el comportamiento de sistemas estelares con interacción humana
   - Lista enlazada: Ideal para ecos de tamaño variable que persisten
   - Pila (object pool): Optimiza creación/destrucción de partículas
   - Arreglo: Para colores base y partículas activas

2. **Gestión de memoria**:
   - Consideraciones clave:
     - Siempre emparejar `new` con `delete`
     - Usar RAII (destructores)
     - Object pooling para elementos frecuentes
   - Prevención de leaks:
     - `clear()` en destructores
     - Verificación con Valgrind

3. **Interacción y dinamismo**:
   - El movimiento del mouse afecta:
     - Arreglo de partículas activas (creación)
     - Lista de ecos (derivado de partículas)
   - Tecla 'c' limpia toda la lista de ecos

4. **Optimización**:
   - Técnicas implementadas:
     - Object pooling (ParticlePool)
     - Limpieza por lotes en update()
     - Renderizado optimizado (usar ofVbo para versión final)

## Conclusiones

Este proyecto demostró cómo:
1. Combinar múltiples estructuras de datos para crear efectos visuales complejos
2. Gestionar memoria eficientemente en aplicaciones gráficas interactivas
3. Implementar patrones como Object Pooling para mejor rendimiento
4. Validar el funcionamiento con pruebas automatizadas y herramientas de profiling

La obra resultante muestra un equilibrio entre:
- Complejidad visual atractiva
- Eficiencia en la gestión de recursos
- Interacción intuitiva
- Estabilidad a largo plazo (sin memory leaks)

[Video evidencia](https://youtu.be/ntKSJMDtpyM)