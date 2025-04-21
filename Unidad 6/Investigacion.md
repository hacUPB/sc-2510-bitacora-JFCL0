# Sesion 1: Profundizacion teorica de patrones de diseño

A continuación, te presentaré los recursos teóricos para cada patrón. Estudia cada uno detenidamente y conversa con ChatGPT sobre ellos. Puedes hacer preguntas, pedir ejemplos y aclarar tus dudas.

- [Factory method](https://refactoring.guru/design-patterns/factory-method).  
El objetivo de esta estructura de datos es crear una clase madre, que crea productos. Esta clase la llamaremos "Creador madre", y al producto que crea, "producto madre", que desarrolla una **interface**. A partir de la clase "Creador madre" se crean clases derivadas, encargadas de crear productos derivados. Por ejemplo, digamos, se crea un "Creador secundario 1" y un "Creador secundario 2". Estos creadores se conectan con los derivados del "producto madre", que digamos en este caso seria "producto derivado 1" y "producto derivado 2". La funcion del "Creador secundario 1" es **especificamente** la creacion de "productos derivados 1", siendo estas **interfaces especificas segun su necesidad**. Esto permite que de darse la necesidad de agregar partes al programa con las que no se contaba previamente, estas se puedan hacer de forma eficiente, **sin tener que modificar el resto del codigo escrito previamente**.
- [Observer](https://refactoring.guru/design-patterns/observer).  
Se refiere a una estructura de diseño de interface con un objeto madre digamos le llamamos **editor**, desde este editor se crea un **administrador de eventos**, el cual registrara a los usuarios registrados a la pagina inicial, y manejara los metodos que permiten dar estos usuarios un titulo adicional de **suscriptor**, removerlo, y el proceso desde el cual se **comunicara a la clase heredada de esta de algun evento**. A partir de esta clase secundaria **administrador de eventos** se heredan la clase de, digamos, **suscriptor**, que tendria un metodo para poder actualizar desde la informacion enviada en el metodo **actualizar** de la clase **administrador de eventos**. Desde esta clase final se pueden crear mas clases hijo que hagan de el proceso de **actualizar** en algo mas especifico dependiendo de las **necesidades por las cuales se ha creado una subclase suscriptor en primer lugar**. De esta manera, en caso de requerirse agregar mas especificaciones para nuevos tipos de **suscriptores**, redactarla **no afectaria la clase principal ni ninguna parte del codigo previamente escrito**.
- [State](https://refactoring.guru/design-patterns/state).  
Esta estructura busca evitar que el codigo dependa de condicionales y switches, esto mediante generar la siguiente estructura de clases: Primero se crea una **clase madre**, la cual almacenara los **estados** que requiera el programa. Esta **clase madre** contiene los metodos iniciales que cada sub-clase derivada de **estados** adecuara a sus propios requerimientos. A partir de estas clases de **estados** se puede generar estados mas especificos en caso de requerirlo, de forma que agregar y ejecutar sus respectivos metodos se haga de manera mas eficiente, y que en caso de requerir editar una clase o estado no implique mover partes diferentes del codigo con el fin de lograrlo.

# Sesión 2: análisis de un caso de estudio

Ahora te voy a presentar un caso de estudio que utiliza los patrones de diseño que hemos estudiado. Ten presente que no voy a partir el código en múltiples archivos para que puedas hacer las pruebas y configuración rápidamente, pero en la práctica deberías hacerlo por orden y para favorecer el trabajo en equipo.

## ofApp.h:

```cpp
#pragma once

#include "ofMain.h"
#include <vector>
#include <string>

class Observer {
public:
    virtual void onNotify(const std::string& event) = 0;
};

class Subject {
public:
    void addObserver(Observer* observer);
    void removeObserver(Observer* observer);
protected:
    void notify(const std::string& event);
private:
    std::vector<Observer*> observers;
};

class Particle;

class State {
public:
    virtual void update(Particle* particle) = 0;
    virtual void onEnter(Particle* particle) {}
    virtual void onExit(Particle* particle) {}
    virtual ~State() = default;
};

class Particle : public Observer { //hace que Particle herede de observer
public:
    Particle();
    ~Particle();

    void update();
    void draw();
    void onNotify(const std::string& event) override;
    void setState(State* newState);

    ofVec2f position;
    ofVec2f velocity;
    float size;
    ofColor color;

private:
    State* state;
};


class NormalState : public State {
public:
    void update(Particle* particle) override;
    virtual void onEnter(Particle* particle) override;
};


class AttractState : public State {
public:
    void update(Particle* particle) override;
};

class RepelState : public State {
public:
    void update(Particle* particle) override;
};

class StopState : public State {
public:
    void update(Particle* particle) override;
};

class ParticleFactory {
public:
    static Particle* createParticle(const std::string& type);
};

class ofApp : public ofBaseApp, public Subject {
    public:
        void setup();
        void update();
        void draw();
        void keyPressed(int key);
private:
    std::vector<Particle*> particles;
};
```

## ofApp.cpp

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
    // Inicializar propiedades
    position = ofVec2f(ofRandomWidth(), ofRandomHeight()); //asigna a la particula una posicion aleatoria en x y y
    velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f));//asigna a la particula una velocidad aleatoria entre -0.5f y 0.5f tanto en x como en y
    size = ofRandom(2, 5); //asigna a la particula un tamaño aleatorio entre 2 y 5
    color = ofColor(255); //asigna a la particula que su color este completamente saturado

    state = new NormalState(); //inicializa la particula en estado Normal.
}

Particle::~Particle() { //destructor de particulas
    delete state;
}

void Particle::setState(State* newState) {
    if (state != nullptr) {
        state->onExit(this); //ejecuta virtual void State::onExit de la clase state para la particula
        delete state; //Elimina el estado que estaba ahi previamente
    }
    state = newState; //Crea un nuevo estado
    if (state != nullptr) {
        state->onEnter(this); //Ejecuta virtual void State::onEnter de la clase state para la particula
    }
}

void Particle::update() {
    if (state != nullptr) {
        state->update(this); //Ejecuta el metodo de state::update para la particula, dependiendo del state almacenado dentro de esta
    }
    // Mantener las partículas dentro de la ventana
    if (position.x < 0 || position.x > ofGetWidth()) velocity.x *= -1;
    if (position.y < 0 || position.y > ofGetHeight()) velocity.y *= -1;
}

void Particle::draw() {
    ofSetColor(color); //aplica a el circulo que va a dibujar el color almacenado en la particula
    ofDrawCircle(position, size); //dibuja el circulo aplicando la posicion y tamaño almacenado en la particula
}

void Particle::onNotify(const std::string& event) {
    if (event == "attract") {
        setState(new AttractState()); //recibe el evento enviado por el metodo ofApp::keypressed y hace que los observers contenidos por Subject, y en consecuencia, las particulas contenidas en la vtable de la funcion Observer::onNotify cambien su atributo state a AttractState
    }
    else if (event == "repel") {
        setState(new RepelState()); //recibe el evento enviado por el metodo ofApp::keypressed y hace que los observers contenidos por Subject, y en consecuencia, las particulas contenidas en la vtable de la funcion Observer::onNotify cambien su atributo state a RepelState
    }
    else if (event == "stop") {
        setState(new StopState()); //recibe el evento enviado por el metodo ofApp::keypressed y hace que los observers contenidos por Subject, y en consecuencia, las particulas contenidas en la vtable de la funcion Observer::onNotify cambien su atributo state a StopState
    }
    else if (event == "normal") {
        setState(new NormalState()); //recibe el evento enviado por el metodo ofApp::keypressed y hace que los observers contenidos por Subject, y en consecuencia, las particulas contenidas en la vtable de la funcion Observer::onNotify cambien su atributo state a NormalState
    }
}

void NormalState::update(Particle* particle) { //Si se activa el metodo void State::update(), por polimorfismo, si su state == NormalState, este ejecutara esta version.
    particle->position += particle->velocity; 
}

void NormalState::onEnter(Particle* particle) {
    particle->velocity = ofVec2f(ofRandom(-0.5f, 0.5f), ofRandom(-0.5f, 0.5f)); //Si se activa el metodo virtual void State::onEnter(), por polimorfismo, si su state == NormalState, este ejecutara esta version.
}

void AttractState::update(Particle* particle) { //Si se activa el metodo void State::update(), por polimorfismo, si su state == AttractState, este ejecutara esta version.
    ofVec2f mousePosition(((ofApp*)ofGetAppPtr())->mouseX, ((ofApp*)ofGetAppPtr())->mouseY);
    ofVec2f direction = mousePosition - particle->position;
    direction.normalize();
    particle->velocity += direction * 0.05;
    ofClamp(particle->velocity.x, -3, 3);
    particle->position += particle->velocity * 0.2;
}

void RepelState::update(Particle* particle) { //Si se activa el metodo void State::update(), por polimorfismo, si su state == RepelState, este ejecutara esta version.
    ofVec2f mousePosition(((ofApp*)ofGetAppPtr())->mouseX, ((ofApp*)ofGetAppPtr())->mouseY);
    ofVec2f direction = particle->position - mousePosition;
    direction.normalize();
    particle->velocity += direction * 0.05;
    ofClamp(particle->velocity.x, -3, 3);
    particle->position += particle->velocity * 0.2;
}

void StopState::update(Particle* particle) { //Si se activa el metodo void State::update(), por polimorfismo, si su state == StopState, este ejecutara esta version.
    particle->velocity.x = 0;
    particle->velocity.y = 0;
}

Particle* ParticleFactory::createParticle(const std::string& type) {
    Particle* particle = new Particle();

    if (type == "star") { //si la particula se tipifica como star se le dan esos parametros para el atributo size y color
        particle->size = ofRandom(2, 4);
        particle->color = ofColor(255, 0, 0);
    }
    else if (type == "shooting_star") { //si la particula se tipifica como shooting_star se le dan esos parametros para el atributo size, color y velocidad
        particle->size = ofRandom(3, 6);
        particle->color = ofColor(0, 255, 0);
        particle->velocity *= 3;
    }
    else if (type == "planet") { //si la particula se tipifica como planet se le dan esos parametros para el atributo size y color
        particle->size = ofRandom(5, 8);
        particle->color = ofColor(0, 0, 255);
    }
    return particle; //devuelve la particula creada a la funcion ofApp::setup()
}


void ofApp::setup() {
    ofBackground(0);
    // Crear partículas usando la fábrica
    for (int i = 0; i < 100; ++i) {
        Particle* p = ParticleFactory::createParticle("star"); //genera 100 particulas de tipo star
        particles.push_back(p); //Almacena las particulas en memoria heap
        addObserver(p); //crea observador que creara la vtable que almacenara los punteros de las particulas creadas
    }

    for (int i = 0; i < 5; ++i) {
        Particle* p = ParticleFactory::createParticle("shooting_star"); //genera 5 particulas de tipo shooting_star
        particles.push_back(p);//Almacena las particulas en memoria heap
        addObserver(p); //crea observador que creara la vtable que almacenara los punteros de las particulas creadas
    }

    for (int i = 0; i < 10; ++i) {
        Particle* p = ParticleFactory::createParticle("planet"); //genera 10 particulas de tipo planet
        particles.push_back(p);//Almacena las particulas en memoria heap
        addObserver(p);//crea observador que creara la vtable que almacenara los punteros de las particulas creadas
    }

}


void ofApp::update() {
    for (Particle* p : particles) {
        p->update(); //ejecuta el void update para cada particula almacenada en el vector ofApp::std::vector<Particle*> particles;
    }
}


void ofApp::draw() {
    for (Particle* p : particles) {
        p->draw(); //ejecuta el void draw para cada particula almacenada en el vector ofApp::std::vector<Particle*> particles;
    }
}

void ofApp::keyPressed(int key) {
    if (key == 's') {
        notify("stop"); //en caso de presionarse la tecla s, ejecuta el metodo Subject::notify con el evento "stop"
    }
    else if (key == 'a') {
        notify("attract"); //en caso de presionarse la tecla a, ejecuta el metodo Subject::notify con el evento "attract"
    }
    else if (key == 'r') {
        notify("repel"); //en caso de presionarse la tecla r, ejecuta el metodo Subject::notify con el evento "repel"
    }
    else if (key == 'n') {
        notify("normal"); //en caso de presionarse la tecla n, ejecuta el metodo Subject::notify con el evento "notify"
    }
}
```

### Ahora te pediré que te tomes un tiempo para analizar el código y entender su funcionamiento.

- ¿Qué hace el patrón observer en este caso?  
El patron observer tiene un metodo `virtual void onNotify()`. Este se conecta con un void principal `ofApp::keyPressed(int key)`, el cual dependiendo de cual tecla se presione enviara un `event` al `void Subject::notify(const std::string& event)`, en el cual finalmente se hara el llamado a `observer->onNotify(event)`, en el cual se contiene una serie de condicionales, con los cuales, dependiendo del evento que se envie, cambiaran el atributo `state` de cada particula contenida en la **vtable** de este metodo, sin embargo, como se realiza a partir de una orden de `Subject` entonces cada particula almacenada en cada **patron observer** se vera afectada. Teniendo esto en cuenta, la funcion de la clase **observer** es puramente **contener los punteros de cada Particle almacenada en el, para poder ejecutar el metodo `virtual void onNotify(event)` en cada una de las particulas que almacena.**
- ¿Qué hace el patrón factory en este caso?  
El patron factory contiene un unico metodo `static Particle* createParticle(const std::string& type)`. Este se conecta con un void principal `ofApp::Setup()`, en el cual se indica el tipo de particula que se quiere crear y llama al metodo de la siguiente manera: `Particle* p = ParticleFactory::createParticle(const std::string& type)`, a partir de esta se pone a funcionar el unico metodo de esta clase. Este contiene multiples condicionales, especificamente, en torno a los tipos **star**, **shooting_star** y **planet**. Segun estos tipos se le otorga a la particula que se esta creando distintos valores para sus respectivos **atributos**. Teniendo esto en cuenta, la funcion de la clase **ParticleFactory** es puramente de **crear particulas** segun el tipo que le es indicado desde el metodo `ofApp::Setup()`.
- ¿Qué hace el patrón state en este caso?  
El patron State funciona como una **clase madre**, generando distintos metodos, siendo estos `virtual void update(Particle* particle = 0)`, `virtual void onEnter(Particle* particle)`, `void onExit(Particle* particle)` y su respectivo destructor. Su funcionalidad en este programa es poder generar **ordenes**, y a partir de las respectivas **vtables** de cada uno de sus metodos, asegurar que independientemente del estado de la particula que este afectando, este cumpla el **override** de la orden dada por esta clase para esta. Las clases que heredan de este metodo son `NormalState` (que tiene un override adicional al metodo `void State::onEnter(Particle* particle)`), `AttractState`, `RepelState` y `StopState`, que todas estas comparten un override al metodo `void State::update(Particle* particle)`.

### Experimenta con el código y realiza algunas modificaciones para entender mejor su funcionamiento. Por ejemplo:

- Adiciona un nuevo tipo de partícula.  
**Partes modificadas**: `ofApp::Setup()` y `Particle* ParticleFactory::createParticle(const std::string& type)`.  
Este cambio agrega un nuevo tipo de particula `dot`. Su tamaño es forzosamente de **1**, su color es **blanco** y su velocidad es **la mitad de la velocidad del resto de particulas**.
```cpp
Particle* ParticleFactory::createParticle(const std::string& type) {
    Particle* particle = new Particle();

    if (type == "star") { 
        particle->size = ofRandom(2, 4);
        particle->color = ofColor(255, 0, 0);
    }
    else if (type == "shooting_star") { 
        particle->size = ofRandom(3, 6);
        particle->color = ofColor(0, 255, 0);
        particle->velocity *= 3;
    }
    else if (type == "planet") { 
        particle->size = ofRandom(5, 8);
        particle->color = ofColor(0, 0, 255);
    }
    else if (type == "dot") { //parte adicionada
        particle->size = 1;
        particle->color = ofColor(255);
        particle->velocity *= 0.5;
    }
    return particle; 
}


void ofApp::setup() {
    ofBackground(0);
    
    for (int i = 0; i < 100; ++i) {
        Particle* p = ParticleFactory::createParticle("star"); 
        particles.push_back(p); 
        addObserver(p); 
    }

    for (int i = 0; i < 5; ++i) {
        Particle* p = ParticleFactory::createParticle("shooting_star"); 
        particles.push_back(p);
        addObserver(p); 
    }

    for (int i = 0; i < 10; ++i) {
        Particle* p = ParticleFactory::createParticle("planet"); 
        particles.push_back(p);
        addObserver(p);
    }
    for (int i = 0; i < 200; i++) { // parte adicionada
        Particle* p = ParticleFactory::createParticle("dot");
        particles.push_back(p);
        addObserver(p);
    }

}
```
- Adiciona un nuevo estado.  
**Partes modificadas**: Archivo `ofApp.h`: Clase `FastState:: public State` agregada, Archivo `ofApp.cpp`: `void ofApp::keyPressed(int key)` modificado, `void Particle::onNotify(const std::string& event)` modificado, `void FastState::update(Particle* particle)` agregado.  
Este nuevo estado busca incrementar considerablemente la velocidad de movimiento de las particulas.

```cpp
//---------------------------------------------------------------------
//ofApp.h
//---------------------------------------------------------------------

class FastState : public State { //agregado
public: 
    void update(Particle* particle) override;
};

//---------------------------------------------------------------------
//ofApp.cpp
//---------------------------------------------------------------------


void Particle::onNotify(const std::string& event) {
    if (event == "attract") {
        setState(new AttractState()); 
    }
    else if (event == "repel") {
        setState(new RepelState()); 
    }
    else if (event == "stop") {
        setState(new StopState()); 
    }
    else if (event == "normal") {
        setState(new NormalState()); 
    }
    else if (event == "fast") { //agregado
        setState(new FastState());
    }
}

//---------------------------------------------------------------------

void FastState::update(Particle* particle) { //agregado
    particle->velocity.x *= 3;
    particle->velocity.y *= 3;
}

//---------------------------------------------------------------------

void ofApp::keyPressed(int key) {
    if (key == 's') {
        notify("stop"); 
    }
    else if (key == 'a') {
        notify("attract"); 
    }
    else if (key == 'r') {
        notify("repel"); 
    }
    else if (key == 'n') {
        notify("normal"); 
    }
    else if (key == 'f') { //agregado
        notify("fast");
    }
}
```
- Modifica el comportamiento de las partículas.  
**Partes modificadas**: `Particle* ParticleFactory::createParticle(const std::string& type)` modificado, `void Particle::draw()` modificado.  
El cambio busca cambiar los parametros de creacion de particulas y el como se dibujan, afectando su comportamiento.
```cpp
void Particle::draw() {
    ofSetColor(color); 
    ofDrawTriangle(position.x - size/2, position.y + size/2, position.x + size/2, position.y + size/2, position.x, position.y - size/2); //modificado
}

//---------------------------------------------------------------------

Particle* ParticleFactory::createParticle(const std::string& type) {
    Particle* particle = new Particle();

    if (type == "star") { 
        particle->size = ofRandom(10, 20); //modificado
        particle->color = ofColor(255, 255, 0); //modificado
    }
    else if (type == "shooting_star") { 
        particle->size = ofRandom(40, 60); //modificado
        particle->color = ofColor(0, 255, 255); //modificado
        particle->velocity *= 14; //modificado
    }
    else if (type == "planet") { 
        particle->size = ofRandom(5, 16); //modificado
        particle->color = ofColor(255, 0, 255); //modificado
    }
    
    return particle; 
}
```
- Crea otros eventos para notificar a las partículas.  

- Adiciona un nuevo estado.  
**Partes modificadas**: Archivo `ofApp.h`: Clase `FastState:: public State` agregada, Archivo `ofApp.cpp`: `void ofApp::keyPressed(int key)` modificado, `void Particle::onNotify(const std::string& event)` modificado, `void FastState::update(Particle* particle)` agregado.  
Este nuevo estado busca incrementar considerablemente la velocidad de movimiento de las particulas. **Si, es el mismo que en el punto de agregar un estado.**

```cpp
//---------------------------------------------------------------------
//ofApp.h
//---------------------------------------------------------------------

class FastState : public State { //agregado
public: 
    void update(Particle* particle) override;
};

//---------------------------------------------------------------------
//ofApp.cpp
//---------------------------------------------------------------------


void Particle::onNotify(const std::string& event) {
    if (event == "attract") {
        setState(new AttractState()); 
    }
    else if (event == "repel") {
        setState(new RepelState()); 
    }
    else if (event == "stop") {
        setState(new StopState()); 
    }
    else if (event == "normal") {
        setState(new NormalState()); 
    }
    else if (event == "fast") { //agregado
        setState(new FastState());
    }
}

//---------------------------------------------------------------------

void FastState::update(Particle* particle) { //agregado
    particle->velocity.x *= 3;
    particle->velocity.y *= 3;
}

//---------------------------------------------------------------------

void ofApp::keyPressed(int key) {
    if (key == 's') {
        notify("stop"); 
    }
    else if (key == 'a') {
        notify("attract"); 
    }
    else if (key == 'r') {
        notify("repel"); 
    }
    else if (key == 'n') {
        notify("normal"); 
    }
    else if (key == 'f') { //agregado
        notify("fast");
    }
}
```

# Sesión 3: revisión de la teoría y caso de estudio

En esta sesión de trabajo por fuera de aula continuarás el proceso de análisis y experiementación.  

R/ 👍