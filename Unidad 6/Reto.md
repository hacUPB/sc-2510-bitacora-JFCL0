# ofApp.h
```cpp
#pragma once

#include "ofMain.h"
#include <vector>
#include <string>

class Observer { //implementacion de patron de diseño Observer
public:
	virtual void onNotify(const std::string& event) = 0;
};

class Subject {
public:
	void addObserver(Observer* observer);
	void removeObserver(Observer* observer);
protected:
	void notify(const std::string& event); //Facilitar proceso de notificar
private:
	std::vector<Observer*> observers; //Vector de punteros a la direccion de distintos observers.

};

class Particle;

class State { //implementacion de patron de diseño State

public:

	virtual void update(Particle* particle) = 0;
	virtual void onEnter(Particle* particle) {}
	virtual void onExit(Particle* particle) {}
	virtual ~State() = default;
};

class Particle : public Observer {
public:
	Particle(); //creador
	~Particle(); //destructor

	void update();
	void draw();
	void onNotify(const std::string& event) override;
	void setState(State* newState);

	ofVec2f position;
	ofVec2f velocity;
	ofVec2f acceleration;
	float size;
	float baseAlpha;
	float currentAlpha;
	float flickerSpeed;
	float flickerAmount;
	float flickerOffset;
	ofColor color;

private:
	State* state;
};

class NormalState : public State {
public:
	void update(Particle* particle) override;
	void onEnter(Particle* particle) override;

};

class AccelerationState : public State { //se busca que la velocidad de la particula incremente a partir de su aceleracion.
public:
	void update(Particle* particle) override;

};

class DeaccelerationState : public State {
public:
	void update(Particle* particle) override;
};

class FasterAlphaState : public State { //se busca que incremente la tasa de cambio de alpha.
public:
	void update(Particle* particle) override;

};

class SlowerAlphaState : public State { //se busca que reduzca la tasa de cambio de alpha.
public:
	void update(Particle* particle) override;

};

class StopState : public State { //se busca que detenga completamente las particulas.
public:
	void update(Particle* particle) override;

};

class ParticleFactory { //implementacion de patron de diseño Factory Method
public:
	static Particle* createParticle(const std::string& type);
};

class ofApp : public ofBaseApp, public Subject {
public:
	void setup();
	void draw();
	void update();
	void keyPressed(int key);
private:
	std::vector<Particle*> particles;
};
```
# ofApp.cpp

```cpp
#include "ofApp.h"

void Subject::addObserver(Observer* observer) {
    observers.push_back(observer); //Almacena puntero de observer a memoria heap.
}

void Subject::removeObserver(Observer* observer) {
    observers.erase(std::remove(observers.begin(), observers.end(), observer), observers.end()); //elimina todos los observers.
}

void Subject::notify(const std::string& event) {
    for (Observer* observer : observers) {
        observer->onNotify(event); //hace que por cada observer en el vector almacenado en el vector de la clase Subject se ejecute el metodo onNotify(), enviando el marcador de evento como indicado por el metodo ofApp::keyPressed
    }
}

Particle::Particle() {
    //inicializar propiedades
    position = ofVec2f(ofRandomWidth(), ofRandomHeight());
    velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
    acceleration = ofVec2f(ofRandom(1.001f, 1.003f), ofRandom(1.001f,1.003f));
    size = ofRandom(2, 5);
    baseAlpha = ofRandom(0, 255);
    currentAlpha = 255;
    flickerSpeed = (5.0f);
    flickerAmount = (0.3f);
    flickerOffset = (ofRandom(TWO_PI));

    state = new NormalState();
}

Particle::~Particle() {
    delete state;
}

void Particle::setState(State* newState) {
    if (state != nullptr) {
        state->onExit(this);
        delete state;
    }
    state = newState;
    if (state != nullptr) {
        state->onEnter(this);
    }
}

void Particle::update() {
    if (state != nullptr) {
        state->update(this);
    }
    // Mantener las particulas dentro de la ventana
    if (position.x < 0 || position.x > ofGetWidth()) velocity.x *= -1;
    if (position.y < 0 || position.y > ofGetHeight()) velocity.y *= -1;
    if (currentAlpha = 0) flickerSpeed *= -1;
    if (currentAlpha = 255) flickerSpeed *= -1;

}

void Particle::draw() {
    ofSetColor(color);
    ofDrawCircle(position, size);
}

void Particle::onNotify(const std::string& event) {
    if (event == "Normal") {
        setState(new NormalState());
    }
    else if (event == "Acceleration") {
        setState(new AccelerationState());
    }
    else if (event == "Deacceleration") {
        setState(new DeaccelerationState());
    }
    else if (event == "FasterAlpha") {
        setState(new FasterAlphaState());
    }
    else if (event == "SlowerAlpha") {
        setState(new SlowerAlphaState());
    }
    else if (event == "Stop") {
        setState(new StopState());
    }
}

void NormalState::update(Particle* particle) {
    particle->position += particle->velocity;
    float time = ofGetElapsedTimef(); 
    float noise = ofNoise(time * 0.5, particle->flickerOffset) * 0.5 + 0.5; 
    float oscillation = sin(time * particle->flickerSpeed + particle->flickerOffset); 
    particle->currentAlpha = particle->baseAlpha * (1.0 - particle->flickerAmount * (0.5 + 0.5 * oscillation) * noise);
    particle->currentAlpha = ofClamp(particle->currentAlpha, 0, particle->baseAlpha);
    particle->color.a = particle->currentAlpha;
}

void NormalState::onEnter(Particle* particle) {
    particle->velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));
}

void AccelerationState::update(Particle* particle) {
    particle->velocity *= particle->acceleration;
    particle->position += particle->velocity;
    float time = ofGetElapsedTimef();
    float noise = ofNoise(time * 0.5, particle->flickerOffset) * 0.5 + 0.5;
    float oscillation = sin(time * particle->flickerSpeed + particle->flickerOffset);
    particle->currentAlpha = particle->baseAlpha * (1.0 - particle->flickerAmount * (0.5 + 0.5 * oscillation) * noise);
    particle->currentAlpha = ofClamp(particle->currentAlpha, 0, particle->baseAlpha);
    particle->color.a = particle->currentAlpha;
}

void DeaccelerationState::update(Particle* particle) {
    particle->velocity /= particle->acceleration;
    particle->position += particle->velocity;
    float time = ofGetElapsedTimef();
    float noise = ofNoise(time * 0.5, particle->flickerOffset) * 0.5 + 0.5;
    float oscillation = sin(time * particle->flickerSpeed + particle->flickerOffset);
    particle->currentAlpha = particle->baseAlpha * (1.0 - particle->flickerAmount * (0.5 + 0.5 * oscillation) * noise);
    particle->currentAlpha = ofClamp(particle->currentAlpha, 0, particle->baseAlpha);
    particle->color.a = particle->currentAlpha;
}

void FasterAlphaState::update(Particle* particle) {
    particle->position += particle->velocity;
    particle->flickerSpeed *= 1.005;
    float time = ofGetElapsedTimef();
    float noise = ofNoise(time * 0.5, particle->flickerOffset) * 0.5 + 0.5;
    float oscillation = sin(time * particle->flickerSpeed + particle->flickerOffset);
    particle->currentAlpha = particle->baseAlpha * (1.0 - particle->flickerAmount * (0.5 + 0.5 * oscillation) * noise);
    particle->currentAlpha = ofClamp(particle->currentAlpha, 0, particle->baseAlpha);
    particle->color.a = particle->currentAlpha;
}

void SlowerAlphaState::update(Particle* particle) {
    particle->position += particle->velocity;
    particle->flickerSpeed *= 0.995;
    float time = ofGetElapsedTimef();
    float noise = ofNoise(time * 0.5, particle->flickerOffset) * 0.5 + 0.5;
    float oscillation = sin(time * particle->flickerSpeed + particle->flickerOffset);
    particle->currentAlpha = particle->baseAlpha * (1.0 - particle->flickerAmount * (0.5 + 0.5 * oscillation) * noise);
    particle->currentAlpha = ofClamp(particle->currentAlpha, 0, particle->baseAlpha);
    particle->color.a = particle->currentAlpha;
}

void StopState::update(Particle* particle) {
    particle->velocity.x = 0;
    particle->velocity.y = 0;
}

Particle* ParticleFactory::createParticle(const std::string& type) {
    Particle* particle = new Particle();

    if (type == "fading_shape") {
        particle->size = ofRandom(10, 14);
        particle->color = ofColor(255, 255, 0, 254);
        particle->flickerSpeed *= 3;
    }
    else if (type == "fast_shape") {
        particle->size = ofRandom(5, 9);
        particle->color = ofColor(0, 255, 255, 254);
        particle->velocity *= 3;

    }
    else if (type == "accelerating_shape") {
        particle->size = ofRandom(2, 4);
        particle->color = ofColor(255, 0, 255, 254);
        particle->acceleration *= 1.01;
    }
    return particle;
}

void ofApp::setup() {
    ofBackground(0);
    // Crear partículas usando la fábrica
    for (int i = 0; i < 100; ++i) {
        Particle* p = ParticleFactory::createParticle("fading_shape");
        particles.push_back(p);
        addObserver(p);
    }

    for (int i = 0; i < 5; ++i) {
        Particle* p = ParticleFactory::createParticle("fast_shape");
        particles.push_back(p);
        addObserver(p);
    }

    for (int i = 0; i < 10; ++i) {
        Particle* p = ParticleFactory::createParticle("accelerating_shape");
        particles.push_back(p);
        addObserver(p);
    }

}
void ofApp::update() {
    for (Particle* p : particles) {
        p->update();
    }
}


void ofApp::draw() {
    for (Particle* p : particles) {
        p->draw();
    }
}

void ofApp::keyPressed(int key) {
    if (key == '1') {
        notify("Normal");
    }
    else if (key == '2') {
        notify("Acceleration");
    }
    else if (key == '3') {
        notify("Deacceleration");
    }
    else if (key == '4') {
        notify("FasterAlpha");
    }
    else if (key == '5') {
        notify("SlowerAlpha");
    }
    else if (key == '6') {
        notify("Stop");
    }
}
```

# RAE1, explicacion de para que y como se implemento cada patron.

Este aplicativo busca generar particulas con diferentes comportamientos. Estos se logran de distintas maneras, veamos primero el motivo detras de cada patron de diseño.

### Observer

Este es utilizado con el fin de poder generar **notificaciones** de forma efectiva. Estas son utilizadas de manera que permitan al usuario interactuar con el programa utilizando el metodo `ofApp::keyPressed(int key)`, llevando asi a un metodo madre en la clase Subject, que hace el llamado al unico metodo de la clase **Observer**, `onNotify(const std::string& event)`. El uso de los **eventos** se da para poder generar un condicional que revise puntualmente **cual es el evento que se registro** y a partir de esto poder tomar una accion **en cada particula almacenada en la `vtable` del metodo**.

### Factory method

Este es utilizado para poder resumir una multitud de lineas de codigo repetitivas en **un solo metodo**, siendo este `Particle* ParticleFactory::createParticle(const std::string& type)`. Lo que se busca mediante su uso es que este genere particulas bajo **una serie de parametros constante**, y pudiendo separar estos parametros en **tipos**, de manera que en caso de requerirse agregar otro tipo, este **no afecte el codigo de los pre-existentes**.  

### State

Este es utilizado para poder generar distintas respuestas del programa en torno a una serie de parametros establecidos en la clase **state**. Esta clase deriva en las siguientes funciones: `Normal (comportamiento default de las particulas)`, `Aceleracion y Desaceleracion (modifica la velocidad segun el parametro aceleracion de la clase particula)`, `FasterAlpha y SlowerAlpha (modifican la velocidad de parpadeo de las particulas)` y `Stop (que detiene completamente el parpadeo y movimiento de las particulas)`. **Utilizar este mecanismo hace de la configuracion de estos comportamientos en algo mas conciso, reduciendo el tiempo de compilacion del programa**. El uso de los states se ve evidenciado en el atributo `State* state` de la clase Particula y en los respectivos metodos de cada sub-state.

[Video evidencia](https://youtu.be/g0nVRHElb5U)