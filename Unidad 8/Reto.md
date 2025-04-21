# **Reto**

En este reto, implementarás una simulación de partículas que se muevan libremente en la pantalla y colisionen entre sí cuando se acerquen. El objetivo es comparar una implementación **secuencial (sin hilos)** y una **concurrente (usando hilos)**, observando las diferencias en rendimiento.

El reto está dividido en dos partes:

## Primera parte: implementación secuencial (sin hilos)

Implementar una simulación donde unas partículas se muevan por la pantalla y colisionen entre sí sin el uso de hilos.

Requisitos:

1. Las partículas deben moverse libremente y rebotar cuando colisionen entre sí.
2. La simulación debe ejecutarse secuencialmente, sin utilizar ofThread ni ninguna técnica de concurrencia.
3. Debes poder aumentar en tiempo real la cantidad de partículas en pantalla, permitiendo observar cómo el rendimiento disminuye al aumentar el número de partículas.
4. Visualizar el **framerate** en pantalla, el número de partículas y el número de colisiones en cada frame.

### ofApp.h

```cpp
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
    void detectCollision(vector<Boid>& boids);
    void setCollisionCooldown(float time) { collisionCooldown = time; }
    float getCollisionCooldown() const { return collisionCooldown; }
    int getCollisionCount() const { return collisionCount; } 

    ofVec2f collisionCalc();
    ofVec2f separate(const vector<Boid>& boids);
    ofVec2f align(const vector<Boid>& boids);
    ofVec2f cohere(const vector<Boid>& boids);

    ofVec2f position;
    ofVec2f velocity;
    ofVec2f acceleration;
    float r;
    float m;
    float maxforce;
    float maxspeed;
    ofVec2f afterCollisionV;
    int collisionCount;

private:
    float collisionCooldown;
    const float COLLISION_COOLDOWN_TIME = 0.5f;
    vector<Boid*> collidedWith;
    bool collisionProcessed;
};

class Flock {
public:
    void addBoid(float x, float y);
    vector<Boid> boids;
};

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();
    void mouseDragged(int x, int y, int button);

    int collisionsThisFrame;
    Flock flock;
};
```

### ofApp.cpp

```cpp
#include "ofApp.h"

Boid::Boid(float x, float y) {
    acceleration.set(0, 0);
    velocity.set(ofRandom(-1, 1), ofRandom(-1, 1));
    position.set(x, y);
    r = 3.0;
    m = ofRandom(1.0, 3.0);
    maxspeed = 3;
    maxforce = 0.05;
    afterCollisionV.set(0, 0);
    collisionCooldown = 0;
    collisionProcessed = false;
}

void Boid::run(const vector<Boid>& boids) {
    detectCollision(const_cast<vector<Boid>&>(boids));
    flock(boids);
    update();
    borders();
}

void Boid::applyForce(ofVec2f force) {
    acceleration += force / m;
}

void Boid::detectCollision(vector<Boid>& boids) {
    collidedWith.clear();
    collisionProcessed = false;
    collisionCount = 0;

    if (collisionCooldown > 0) { // evita choques infinitos
        collisionCooldown -= ofGetLastFrameTime();
        return;
    }

    for (auto& other : boids) {
        if (&other == this) continue; // evita chocarse contra si mismo
        if (other.getCollisionCooldown() > 0) continue;

        float d = position.distance(other.position);
        if (d > 0 && d < r + other.r) { // cuenta colision si la distancia entre el centro de los dos boids es mayor que sus respectivos radios sumados
            collidedWith.push_back(&other); // agrega a cada uno de los 2 boids a la lista de los que aun no se analizan del otro
            collisionCooldown = COLLISION_COOLDOWN_TIME;
            other.setCollisionCooldown(COLLISION_COOLDOWN_TIME); // agrega a cada uno de los 2 boids a la lista de los que aun no se analizan del otro

            // Separación inmediata
            ofVec2f dir = (position - other.position).normalized();
            float overlap = (r + other.r) - d;
            position += dir * overlap * 0.5f;
            other.position -= dir * overlap * 0.5f;
            collisionCount++;
        }
    }
}

void Boid::flock(const vector<Boid>& boids) {
    if (!collidedWith.empty()) { // si hay items en el arreglo de boids cuya colision no se ha analizado, solamente permite ejecutar el metodo de calcular colision
        ofVec2f coC = collisionCalc();
        coC.limit(maxspeed);
        velocity = coC * 0.5f;
        collisionProcessed = true;
    }
    else { // si no hay items en el arreglo de boids cuya colision no se ha analizado permite ejecutar los modificadores de aceleracion (fuerzas)
        ofVec2f sep = separate(boids);
        ofVec2f ali = align(boids);
        ofVec2f coh = cohere(boids);

        sep *= 1.5f;
        applyForce(sep);
        applyForce(ali);
        applyForce(coh);
    }
}

void Boid::update() {
    if (collisionProcessed) { //si se proceso cualquier colision en la particula se evitan los calculos de aceleracion y limitacion de velocidad
        collisionProcessed = false;
    }
    else {
        velocity += acceleration;
        velocity.limit(maxspeed);
    }
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
    float width = ofGetWidth();
    float height = ofGetHeight();

    if (position.x < -r) position.x = width + r;
    if (position.y < -r) position.y = height + r;
    if (position.x > width + r) position.x = -r;
    if (position.y > height + r) position.y = -r;
}

void Boid::draw() {
    float theta = atan2(velocity.y, velocity.x) + PI / 2;
    ofSetColor(255);
    if (collisionCooldown > 0) ofSetColor(255, 0, 0); // si el boid esta en cooldown se vera diferente
    ofPushMatrix();
    ofTranslate(position.x, position.y);
    ofRotateRad(theta);
    ofDrawTriangle(0, -r * 2, -r, r * 2, r, r * 2);
    ofPopMatrix();
}

ofVec2f Boid::collisionCalc() {
    if (collidedWith.empty()) return velocity;

    Boid& other = *collidedWith[0]; // se da acceso a los valores de los atributos del vector enlistado como que colisiono con el que esta procesando en el momento.
    ofVec2f n = position - other.position; // obtiene componente normal entre los 2 boids
    n.normalize(); // normaliza n
    ofVec2f t(-n.y, n.x); // obtiene componente tangencial entre los 2 boids

    float v1n = velocity.dot(n); // operacion multiplicacion punto en matrices, vector normal de la velocidad 1
    float v1t = velocity.dot(t); // operacion multiplicacion punto en matrices, vector tangencial de la velocidad 1
    float v2n = other.velocity.dot(n); // operacion multiplicacion punto en matrices, vector normal de la velocidad 2

    float newV1n = (v1n * (m - other.m) + 2 * other.m * v2n) / (m + other.m); // velocidad final normal
    return newV1n * n + v1t * t; // devuelve velocidad final
}

ofVec2f Boid::separate(const vector<Boid>& boids) {
    float desiredSeparation = r * 4;
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
        if (steer.length() > 0) {
            steer.normalize();
            steer *= maxspeed;
            steer -= velocity;
            steer.limit(maxforce);
        }
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
        sum /= (float)count;
        return seek(sum);
    }
    return ofVec2f(0, 0);
}

void Flock::addBoid(float x, float y) {
    boids.emplace_back(x, y);
}

void ofApp::setup() {
    ofSetFrameRate(60);
    flock.boids.reserve(50);
    for (int i = 0; i < 50; i++) {
        flock.addBoid(ofRandomWidth(), ofRandomHeight());
    }
}

void ofApp::update() {
    collisionsThisFrame = 0;
    for (Boid& b : flock.boids) {
        b.run(flock.boids);
        collisionsThisFrame += b.getCollisionCount();
    }
}

void ofApp::draw() {
    ofBackground(0);
    for (Boid& b : flock.boids) {
        b.draw();
    }
    ofDrawBitmapString("FPS: " + ofToString(ofGetFrameRate()), 20, 20);
    ofDrawBitmapStringHighlight("Boids: " + ofToString(flock.boids.size()), 20, 40);
    ofDrawBitmapString("Colisiones este frame: " + ofToString(collisionsThisFrame), 20, 60);

}

void ofApp::mouseDragged(int x, int y, int button) {
    flock.addBoid(x, y);
}
```

## Segunda parte: implementación concurrente (con hilos)

Modificar la simulación para usar varios hilos (ofThread), de manera que el cálculo del movimiento y las colisiones de las partículas esté distribuido entre diferentes hilos.

Requisitos:

1. Las partículas deben estar divididas en 4 grupos y cada grupo deberá ser gestionado por un hilo.
2. No es necesario que aumentes en tiempo real la cantidad de partículas. Distribuye de manera equitativa las partículas en los 4 grupos. De todas maneras debes poder configurar la cantidad de partículas en tu programa para que puedas comparar con la versión sin hilos. NO OLVIDES: esta variación lo realizarás en tiempo de compilación.
3. Las colisiones dentro de un mismo grupo de partículas se deben calcular sin necesidad de sincronización.
4. Las colisiones entre partículas de diferentes grupos deben gestionarse usando sincronización (locks) para evitar condiciones de carrera.
5. Visualizar el **framerate**, el número de partículas, el número de grupos y el número de colisiones en cada frame.

### ofApp.h

```cpp
#pragma once
#include "ofMain.h"
#include <thread>
#include <mutex>
#include <atomic>

class Boid {
public:
    Boid(float x, float y, int groupId);
    void run(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx);
    void applyForce(ofVec2f force);
    void flock(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx);
    void update();
    void borders();
    ofVec2f seek(ofVec2f target);
    void draw();
    void detectCollision(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx);
    void setCollisionCooldown(float time) { collisionCooldown = time; }
    float getCollisionCooldown() const { return collisionCooldown; }
    int getCollisionCount() const { return collisionCount; }
    int getGroupId() const { return groupId; }

    ofVec2f collisionCalc();
    ofVec2f separate(vector<unique_ptr<Boid>>& boids);
    ofVec2f align(vector<unique_ptr<Boid>>& boids);
    ofVec2f cohere(vector<unique_ptr<Boid>>& boids);

    ofVec2f position;
    ofVec2f velocity;
    ofVec2f acceleration;
    float r;
    float m;
    float maxforce;
    float maxspeed;
    ofVec2f afterCollisionV;
    atomic<int> collisionCount;

private:
    float collisionCooldown;
    const float COLLISION_COOLDOWN_TIME = 0.5f;
    vector<Boid*> collidedWith;
    bool collisionProcessed;
    int groupId;
};

class Flock {
public:
    void addBoid(float x, float y, int groupId);
    void run();
    void updateGroup(int groupId);

    vector<unique_ptr<Boid>> boids;
    vector<vector<Boid*>> groups;
    vector<Boid*> allBoids;
    mutex mtx;
    atomic<int> totalCollisions{ 0 };
};

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();
    void mouseDragged(int x, int y, int button);
    void exit();

    Flock flock;
    vector<thread> threads;
    atomic<bool> running{ true };
    const int NUM_GROUPS = 4;
    const int BOIDS_PER_GROUP = 50; // Cambiar este valor para ajustar número de partículas
};
```

### ofApp.cpp

```cpp
#include "ofApp.h"
#include "memory"

Boid::Boid(float x, float y, int groupId) : groupId(groupId) {
    acceleration.set(0, 0);
    velocity.set(ofRandom(-1, 1), ofRandom(-1, 1));
    position.set(x, y);
    r = 3.0;
    m = ofRandom(1.0, 3.0);
    maxspeed = 3;
    maxforce = 0.05;
    afterCollisionV.set(0, 0);
    collisionCooldown = 0;
    collisionProcessed = false;
    collisionCount = 0;
}

void Boid::run(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx) {
    detectCollision(boids, otherGroups, mtx);
    flock(boids, otherGroups, mtx);
    update();
    borders();
}

void Boid::applyForce(ofVec2f force) {
    acceleration += force / m;
}

void Boid::detectCollision(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx) {
    collidedWith.clear();
    collisionProcessed = false;
    collisionCount = 0;

    if (collisionCooldown > 0) {
        collisionCooldown -= ofGetLastFrameTime();
        return;
    }

    // Colisiones dentro del mismo grupo (sin sincronización necesaria)
    for (auto& other : boids) {
        if (other.get() == this) continue;
        if (other->getCollisionCooldown() > 0) continue;

        float d = position.distance(other->position);
        if (d > 0 && d < r + other->r) {
            collidedWith.push_back(other.get());
            collisionCooldown = COLLISION_COOLDOWN_TIME;
            other->setCollisionCooldown(COLLISION_COOLDOWN_TIME);

            ofVec2f dir = (position - other->position).normalized();
            float overlap = (r + other->r) - d;
            position += dir * overlap * 0.5f;
            other->position -= dir * overlap * 0.5f;
            collisionCount++;
        }
    }

    // Colisiones con otros grupos (con sincronización)
    lock_guard<mutex> lock(mtx);
    for (auto other : otherGroups) {
        if (other->getCollisionCooldown() > 0) continue;

        float d = position.distance(other->position);
        if (d > 0 && d < r + other->r) {
            collidedWith.push_back(other);
            collisionCooldown = COLLISION_COOLDOWN_TIME;
            other->setCollisionCooldown(COLLISION_COOLDOWN_TIME);

            ofVec2f dir = (position - other->position).normalized();
            float overlap = (r + other->r) - d;
            position += dir * overlap * 0.5f;
            other->position -= dir * overlap * 0.5f;
            collisionCount++;
        }
    }
}

void Boid::flock(vector<unique_ptr<Boid>>& boids, vector<Boid*>& otherGroups, mutex& mtx) {
    if (!collidedWith.empty()) {
        ofVec2f coC = collisionCalc();
        coC.limit(maxspeed);
        velocity = coC * 0.5f;
        collisionProcessed = true;
    }
    else {
        ofVec2f sep = separate(boids);
        ofVec2f ali = align(boids);
        ofVec2f coh = cohere(boids);

        sep *= 1.5f;
        applyForce(sep);
        applyForce(ali);
        applyForce(coh);
    }
}

void Boid::update() {
    if (collisionProcessed) {
        collisionProcessed = false;
    }
    else {
        velocity += acceleration;
        velocity.limit(maxspeed);
    }
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
    float width = ofGetWidth();
    float height = ofGetHeight();

    if (position.x < -r) position.x = width + r;
    if (position.y < -r) position.y = height + r;
    if (position.x > width + r) position.x = -r;
    if (position.y > height + r) position.y = -r;
}

void Boid::draw() {
    float theta = atan2(velocity.y, velocity.x) + PI / 2;

    // Color diferente por grupo
    ofColor colors[4] = { ofColor::red, ofColor::green, ofColor::blue, ofColor::yellow };
    ofSetColor(colors[groupId % 4]);

    if (collisionCooldown > 0) {
        ofSetColor(255, 255, 255); // Blanco si está en cooldown
    }

    ofPushMatrix();
    ofTranslate(position.x, position.y);
    ofRotateRad(theta);
    ofDrawTriangle(0, -r * 2, -r, r * 2, r, r * 2);
    ofPopMatrix();
}

ofVec2f Boid::collisionCalc() {
    if (collidedWith.empty()) return velocity;

    Boid& other = *collidedWith[0];
    ofVec2f n = position - other.position;
    n.normalize();
    ofVec2f t(-n.y, n.x);

    float v1n = velocity.dot(n);
    float v1t = velocity.dot(t);
    float v2n = other.velocity.dot(n);

    float newV1n = (v1n * (m - other.m) + 2 * other.m * v2n) / (m + other.m);
    return newV1n * n + v1t * t;
}

ofVec2f Boid::separate(vector<unique_ptr<Boid>>& boids) {
    float desiredSeparation = r * 4;
    ofVec2f steer(0, 0);
    int count = 0;

    for (auto& other : boids) {
        float d = position.distance(other->position);
        if (d > 0 && d < desiredSeparation) {
            ofVec2f diff = position - other->position;
            diff.normalize();
            diff /= d;
            steer += diff;
            count++;
        }
    }

    if (count > 0) {
        steer /= (float)count;
        if (steer.length() > 0) {
            steer.normalize();
            steer *= maxspeed;
            steer -= velocity;
            steer.limit(maxforce);
        }
    }
    return steer;
}

ofVec2f Boid::align(vector<unique_ptr<Boid>>& boids) {
    float neighborDist = 50;
    ofVec2f sum(0, 0);
    int count = 0;

    for (auto& other : boids) {
        float d = position.distance(other->position);
        if (d > 0 && d < neighborDist) {
            sum += other->velocity;
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

ofVec2f Boid::cohere(vector<unique_ptr<Boid>>& boids) {
    float neighborDist = 50;
    ofVec2f sum(0, 0);
    int count = 0;

    for (auto& other : boids) {
        float d = position.distance(other->position);
        if (d > 0 && d < neighborDist) {
            sum += other->position;
            count++;
        }
    }

    if (count > 0) {
        sum /= (float)count;
        return seek(sum);
    }
    return ofVec2f(0, 0);
}

void Flock::addBoid(float x, float y, int groupId) {
    boids.push_back(make_unique<Boid>(x, y, groupId));
}

void Flock::updateGroup(int groupId) {
    vector<Boid*> otherGroups;

    // Preparar lista de boids de otros grupos
    for (int i = 0; i < groups.size(); ++i) {
        if (i != groupId) {
            otherGroups.insert(otherGroups.end(), groups[i].begin(), groups[i].end());
        }
    }

    // Actualizar cada boid del grupo
    for (auto boid : groups[groupId]) {
        boid->run(boids, otherGroups, mtx);
    }
}

void Flock::run() {
    totalCollisions = 0;

    // Ejecutar actualización por grupos en paralelo
    vector<thread> threads;
    for (int i = 0; i < groups.size(); ++i) {
        threads.emplace_back(&Flock::updateGroup, this, i);
    }

    // Esperar a que terminen todos los hilos
    for (auto& t : threads) {
        t.join();
    }

    // Contar colisiones totales
    for (auto& boid : boids) {
        totalCollisions += boid->getCollisionCount();
    }
}

void ofApp::setup() {
    ofSetFrameRate(60);

    // Crear boids distribuidos en grupos
    for (int g = 0; g < NUM_GROUPS; g++) {
        flock.groups.emplace_back();
        for (int i = 0; i < BOIDS_PER_GROUP; i++) {
            float x = ofRandomWidth();
            float y = ofRandomHeight();
            flock.addBoid(x, y, g);
        }
    }

    // Organizar punteros a boids por grupos
    int groupIndex = 0;
    for (auto& boid : flock.boids) {
        flock.groups[groupIndex].push_back(boid.get());
        flock.allBoids.push_back(boid.get());
        groupIndex = (groupIndex + 1) % NUM_GROUPS;
    }
}

void ofApp::update() {
    flock.run();
}

void ofApp::draw() {
    ofBackground(0);

    // Dibujar todos los boids
    for (auto& boid : flock.boids) {
        boid->draw();
    }

    // Mostrar estadísticas
    ofSetColor(255);
    ofDrawBitmapString("FPS: " + ofToString(ofGetFrameRate()), 20, 20);
    ofDrawBitmapString("Total Boids: " + ofToString(flock.boids.size()), 20, 40);
    ofDrawBitmapString("Grupos: " + ofToString(NUM_GROUPS), 20, 60);
    ofDrawBitmapString("Colisiones este frame: " + ofToString(flock.totalCollisions), 20, 80);
}

void ofApp::mouseDragged(int x, int y, int button) {
    // Generar random para el grupo al que se unira
    int groupId = ofRandom(0, NUM_GROUPS);

    // agregar el boid
    flock.addBoid(x, y, groupId);

    // crear un puntero al boid
    Boid* newBoid = flock.boids.back().get();

    // agregar puntero al grupo correcto
    flock.groups[groupId].push_back(newBoid);

    // agregarlo a la coleccion general
    flock.allBoids.push_back(newBoid);
}

void ofApp::exit() {
    running = false;
    for (auto& t : threads) {
        if (t.joinable()) t.join();
    }
}
```

## Comparación entre ambas versiones

Después de completar ambas partes, deberás hacer una **comparación de rendimiento** entre la implementación secuencial y la concurrente. Responde a las siguientes preguntas:

1. ¿Cómo afecta el uso de hilos al framerate a medida que aumenta el número de partículas?  
Version sin hilos: Caida de frames desde los 600 boids.  
Version con hilos: Caida de frames desde los 900 boids.  
2. ¿Qué impacto tiene la sincronización (locks) en el rendimiento?  
Hace que para poder acceder a los atributos requeridos para hacer un calculo, los metodos que el computador procesa como threads se vean relentizados, ya que deben esperar a poder acceder a un lock ellos mismos. 
3. ¿Dónde encontraste mayor beneficio en el uso de hilos y dónde causó más complicaciones?  
En el metodo `Flock::run()`, ya que en este se encuentran las ordenes de ejecucion de metodos mas pesados, y permite dedicar nucleos del procesador distintos a estos metodos, de forma que la CPU no tome tanto tiempo de respuesta para darlos.

    [Video evidencia](https://youtu.be/i8JyOrXUj3Y)