# Enunciado

## Archivo ofApp.h

```cpp
#pragma once

#include "ofMain.h"


class Shape {
protected:
    ofPoint position;
    float size;
    ofColor color;

public:
    Shape(ofPoint pos, float s, ofColor c) : position(pos), size(s), color(c) {}
    virtual ~Shape() {}

    
    virtual void draw() = 0;

    
    virtual void update() {
        position.y += ofRandom(-1, 1);
        position.x += ofRandom(-1, 1);
    }
};


class Circle : public Shape {
public:
    Circle(ofPoint pos, float s, ofColor c) : Shape(pos, s, c) {}

    void draw() override {
        ofSetColor(color);
        ofDrawCircle(position, size);
    }

    void update() override {
    Shape::update();
    size *= 0.995f;
    }
};


class Square : public Shape {
public:
    Square(ofPoint pos, float s, ofColor c) : Shape(pos, s, c) {}

    void draw() override {
        ofSetColor(color);
        ofDrawRectangle(position, size, size);
    }
};

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();

    vector<unique_ptr<Shape>> shapes; 
};
```

## Archivo ofApp.cpp

```cpp
#include "ofApp.h"

void ofApp::setup() {
    ofSetBackgroundAuto(false);
    ofSetBackgroundColor(0);
    ofSetFrameRate(60);

    
    for (int i = 0; i < 50; i++) {
        ofPoint pos(ofRandomWidth(), ofRandomHeight());
        float size = ofRandom(10, 40);
        ofColor color(ofRandom(150, 255), ofRandom(150, 255), ofRandom(150, 255), 100);

        if (ofRandom(1) > 0.5) {
            shapes.push_back(make_unique<Circle>(pos, size, color));
        }
        else {
            shapes.push_back(make_unique<Square>(pos, size, color));
        }
    }
}

void ofApp::update() {
    
    for (auto& shape : shapes) {
        shape->update();
    }
}

void ofApp::draw() {
    
    for (auto& shape : shapes) {
        shape->draw();
    }

    
    ofSetColor(0, 0, 0, 10);
    ofDrawRectangle(0, 0, ofGetWidth(), ofGetHeight());
}
```

# Experimentacion

## Archivo ofApp.h
```cpp
#pragma once

#include "ofMain.h"

class Shape {
protected:
    ofPoint position;
    float size;
    ofColor color;

public:
    Shape(ofPoint pos, float s, ofColor c) : position(pos), size(s), color(c) {}
    virtual ~Shape() = default;

    virtual void draw() const = 0; // const para métodos que no modifican el estado
    virtual void update() {
        position.y += ofRandom(-1, 1);
        position.x += ofRandom(-1, 1);
    }
    float getSize();
};

class Circle : public Shape {
public:
    using Shape::Shape; // Hereda el constructor

    void draw() const override {
        ofSetColor(color);
        ofDrawCircle(position, size);
    }

    void update() override {
        Shape::update();
        size *= 0.995f;
        if (size < 1.0f) size = 1.0f; // Evitar tamaños negativos
    }
};

class Square : public Shape {
public:
    using Shape::Shape; // Hereda el constructor

    void draw() const override {
        ofSetColor(color);
        ofDrawRectangle(position, size, size);
    }
};

class ofApp : public ofBaseApp {
public:
    void setup() override;
    void update() override;
    void draw() override;

private:
    vector<unique_ptr<Shape>> shapes;
    ofFbo fbo; // Buffer para dibujo persistente
};
```

## Archivo ofApp.cpp

```cpp
#include "ofApp.h"

void ofApp::setup() {
    ofSetBackgroundAuto(false);
    ofSetBackgroundColor(0);
    ofSetFrameRate(60);

    // Pre-reservar memoria para evitar reubicaciones
    shapes.reserve(100);

    // Crear formas iniciales
    for (int i = 0; i < 50; ++i) { // ++i es ligeramente más eficiente que i++
        ofPoint pos(ofRandomWidth(), ofRandomHeight());
        float size = ofRandom(10, 40);
        ofColor color(ofRandom(150, 255), ofRandom(150, 255), ofRandom(150, 255), 100);

        if (ofRandom(1) > 0.5f) {
            shapes.emplace_back(make_unique<Circle>(pos, size, color));
        }
        else {
            shapes.emplace_back(make_unique<Square>(pos, size, color));
        }
    }

    // Configurar el framebuffer
    fbo.allocate(ofGetWidth(), ofGetHeight());
    fbo.begin();
    ofClear(0, 0, 0, 0);
    fbo.end();
}

void ofApp::update() {
    // Eliminar formas demasiado pequeñas
    shapes.erase(
        remove_if(shapes.begin(), shapes.end(),
            [](const unique_ptr<Shape>& shape) {
                return dynamic_cast<Circle*>(shape.get()) &&
                    static_cast<Circle*>(shape.get())->getSize() < 2.0f;
            }),
        shapes.end()
    );

    // Actualizar formas
    for (auto& shape : shapes) {
        shape->update();
    }
}
float Shape::getSize() {

    return size;
}

void ofApp::draw() {
    // Dibujar en el framebuffer para efecto persistente
    fbo.begin();
    ofSetColor(0, 0, 0, 10);
    ofDrawRectangle(0, 0, ofGetWidth(), ofGetHeight());

    // Dibujar formas
    for (const auto& shape : shapes) {
        shape->draw();
    }
    fbo.end();

    // Dibujar el framebuffer en pantalla
    fbo.draw(0, 0);
}
```

# Reflexion y ajuste final

## Archivo ofApp.h

```cpp
#pragma once

#include "ofMain.h"

class Shape {
protected:
    ofPoint position;
    float size;
    ofColor color;

public:
    Shape(ofPoint pos, float s, ofColor c) : position(pos), size(s), color(c) {}
    virtual ~Shape() = default;

    virtual void draw() const = 0;
    virtual void update() {
        position.y += ofRandom(-1, 1);
        position.x += ofRandom(-1, 1);
    }

    // Mejor implementación de getSize()
    float getSize() const { return size; } // Const y definición inline

    // Nuevo método para verificar si la forma debe ser eliminada
    virtual bool shouldRemove() const { return false; }
};

class Circle : public Shape {
public:
    using Shape::Shape;

    void draw() const override {
        ofSetColor(color);
        ofDrawCircle(position, size);
    }

    void update() override {
        Shape::update();
        size *= 0.995f;
        if (size < 1.0f) size = 1.0f;
    }

    bool shouldRemove() const override {
        return size < 2.0f; // Umbral para eliminación
    }
};

class Square : public Shape {
public:
    using Shape::Shape;

    void draw() const override {
        ofSetColor(color);
        ofDrawRectangle(position, size, size);
    }

    // Los cuadrados podrían tener su propia lógica de eliminación si lo deseas
};

class ofApp : public ofBaseApp {
public:
    void setup() override;
    void update() override;
    void draw() override;
    void windowResized(int w, int h) override; // Nuevo para manejar redimensionamiento
    void keyPressed(int key);

private:
    vector<unique_ptr<Shape>> shapes;
    ofFbo fbo;

    void initializeFbo(); // Función auxiliar para inicializar el FBO
    void addRandomShape(); // Función para añadir formas aleatorias
};
```

## Archivo ofApp.cpp

```cpp
#include "ofApp.h"

void ofApp::initializeFbo() {
    fbo.allocate(ofGetWidth(), ofGetHeight());
    fbo.begin();
    ofClear(0, 0, 0, 0);
    fbo.end();
}

void ofApp::addRandomShape() {
    ofPoint pos(ofRandomWidth(), ofRandomHeight());
    float size = ofRandom(10, 40);
    ofColor color(ofRandom(150, 255), ofRandom(150, 255), ofRandom(150, 255), 100);

    if (ofRandom(1) > 0.5f) {
        shapes.emplace_back(make_unique<Circle>(pos, size, color));
    }
    else {
        shapes.emplace_back(make_unique<Square>(pos, size, color));
    }
}

void ofApp::setup() {
    ofSetBackgroundAuto(false);
    ofSetBackgroundColor(0);
    ofSetFrameRate(60);

    shapes.reserve(100);

    for (int i = 0; i < 50; ++i) {
        addRandomShape();
    }

    initializeFbo();
}

void ofApp::update() {
    // Eliminar formas usando shouldRemove (más limpio y polimórfico)
    shapes.erase(
        remove_if(shapes.begin(), shapes.end(),
            [](const unique_ptr<Shape>& shape) {
                return shape->shouldRemove();
            }),
        shapes.end()
    );

    // Ocasionalmente añadir nuevas formas
    if (ofRandom(1) < 0.05f && shapes.size() < 100) { // Límite máximo de formas
        addRandomShape();
    }

    for (auto& shape : shapes) {
        shape->update();
    }
}

void ofApp::draw() {
    fbo.begin();
    ofSetColor(0, 0, 0, 10);
    ofDrawRectangle(0, 0, ofGetWidth(), ofGetHeight());

    for (const auto& shape : shapes) {
        shape->draw();
    }
    fbo.end();

    fbo.draw(0, 0);

   
}

void ofApp::windowResized(int w, int h) {
    initializeFbo(); // Recrear el FBO al cambiar el tamaño de la ventana
}
void ofApp::keyPressed(int key) { //interaccion de usuario
    if (key == 'e') {
    addRandomShape();
    }
}
```

[Video evidencia](https://youtu.be/kYHz0V8MiGw)