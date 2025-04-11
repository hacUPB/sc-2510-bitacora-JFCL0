## Archivo `app.h`

### Clase Sphere:

Esta clase funciona puramente para la generacion de esferas y su almacenamiento en `glm::vec3`
```cpp
class Sphere {
public:
    Sphere(glm::vec3 pos, float r); //creador de la esfera

    glm::vec3 getPosition() const; //puntero
    float getRadius() const; //puntero
    bool isSelected() const; //boolean para comportamiento cuando se le da click

    void setPosition(glm::vec3 pos); 
    void setRadius(float r); 
    void setSelected(bool s);
    void draw();

private:
    glm::vec3 position;
    float radius;
    bool selected;
};
```

### Clase SphereGrid:
Esta clase es la que da ubicacion y almacena las esferas.
```cpp
class SphereGrid {
public:
    void SphereGrid::setup(int w, int h);
    void generateGrid(int step, float distDiv, float amplitude);
    void SphereGrid::draw();
    
    vector<Sphere>& getSpheres(); //almacena las esferas
    const vector<Sphere>& getSpheres() const;

    int getCurrentStep() const;
    float getCurrentDistDiv() const;
    float getCurrentAmplitude() const; 

private:
    vector<Sphere> spheres;
    int width, height;
    int currentStep;
    float currentDistDiv;
    float currentAmplitude;
};
```

### Clase raycaster:
Esta clase genera el rayo, con el cual se permitira al usuario interactuar con el escenario 3D.
```cpp
class RayCaster {
public:
    static bool intersectSphere(const glm::vec3& rayStart, const glm::vec3& rayDir,
        const Sphere& sphere, glm::vec3& intersection);

    static void mouseToRay(int x, int y, ofEasyCam& cam, glm::vec3& start, glm::vec3& end);
};
```

### Clase SphereSelector:
Esta clase funciona para poder almacenar y diferenciar la esfera que se seleccione mediante el uso del rayo.
```cpp
class SphereSelector {
public:
    
    void mousePressed(int x, int y, ofEasyCam& cam);
    const Sphere* getSelectedSphere() const;

private:
    SphereGrid* grid;
    const Sphere* selectedSphere;
};
```

### Clase ofApp:

Esta clase es directamente la que genera las funcionalidades generales del programa, y desde el cual se genera la camara incluyo aca el `#pragma once` y el `#include "ofMain.h`, ambos necesarios para que todo funcione, y conectandolo a las dependencias incluidas con cada proyecto de **openframeworks**.

```cpp
#pragma once
#include "ofMain.h"

class ofApp : public ofBaseApp {
public:
    void setup();
    void update();
    void draw();

    void keyPressed(int key);
    void mousePressed(int x, int y, int button);

private:
    ofEasyCam cam;
    SphereGrid grid;
    SphereSelector selector;
    ofTrueTypeFont font;
};
```

## Archivo `ofApp.cpp`

### Implementacion de la clase Sphere

```cpp
Sphere::Sphere() : position(0, 0, 0), radius(10.0f), selected(false) {}
Sphere::Sphere(glm::vec3 pos, float r) : position(pos), radius(r), selected(false) {}

void Sphere::draw() {
    ofSetColor(selected ? ofColor::red : ofColor::white);
    ofDrawSphere(position, radius);
}

glm::vec3 Sphere::getPosition() const { return position; } //Funcion puntero
float Sphere::getRadius() const { return radius; } //Funcion puntero
bool Sphere::isSelected() const { return selected; } //Funcion puntero, activa boolean.

void Sphere::setPosition(glm::vec3 pos) { position = pos; }
void Sphere::setRadius(float r) { radius = r; }
void Sphere::setSelected(bool s) { selected = s; }
```

### Implementacion de la clase SphereGrid

```cpp
void SphereGrid::setup(int w, int h) {
    width = w;
    height = h;
    generateGrid(50, 50.0f, 150.0f);
}

void SphereGrid::generateGrid(int step, float distDiv, float amplitude) {
    spheres.clear();
    currentStep = step;
    currentDistDiv = distDiv;
    currentAmplitude = amplitude;

    for (int x = -width / 2; x < width / 2; x += step) { //Funcion posicion de las esferas
        for (int y = -height / 2; y < height / 2; y += step) {
            float z = cos(ofDist(x, y, 0, 0) / distDiv) * amplitude;
            spheres.emplace_back(glm::vec3(x, y, z), 5.0f);
        }
    }
}

void SphereGrid::draw() {
    for (auto& sphere : spheres) {
        sphere.draw();
    }
}

void SphereGrid::clear() { spheres.clear(); }

vector<Sphere>& SphereGrid::getSpheres() { return spheres; }
const vector<Sphere>& SphereGrid::getSpheres() const { return spheres; }

int SphereGrid::getCurrentStep() const { return currentStep; } //Funcion puntero
float SphereGrid::getCurrentDistDiv() const { return currentDistDiv; } //Funcion puntero
float SphereGrid::getCurrentAmplitude() const { return currentAmplitude; } //Funcion puntero
```

### Implementacion de la clase RayCaster
Formulas cortesia de **Deepseek** porque me dan miedo los numeros.
```cpp
bool RayCaster::intersectSphere(const glm::vec3& rayStart, const glm::vec3& rayDir,
    const Sphere& sphere, glm::vec3& intersection) {
    glm::vec3 oc = rayStart - sphere.getPosition();
    float a = glm::dot(rayDir, rayDir);
    float b = 2.0f * glm::dot(oc, rayDir);
    float c = glm::dot(oc, oc) - pow(sphere.getRadius(), 2);

    float discriminant = b * b - 4 * a * c;
    if (discriminant < 0) return false;

    float t = (-b - sqrt(discriminant)) / (2.0f * a);
    intersection = rayStart + t * rayDir;
    return true;
}

void RayCaster::mouseToRay(int x, int y, ofEasyCam& cam, glm::vec3& start, glm::vec3& end) {
    glm::mat4 modelview = cam.getModelViewMatrix();
    glm::mat4 projection = cam.getProjectionMatrix();
    ofRectangle viewport = ofGetCurrentViewport();

    float nx = 2.0f * (x - viewport.x) / viewport.width - 1.0f;
    float ny = 1.0f - 2.0f * (y - viewport.y) / viewport.height;

    glm::vec4 rayStartNDC(nx, ny, -1.0f, 1.0f);
    glm::vec4 rayEndNDC(nx, ny, 1.0f, 1.0f);

    glm::vec4 rayStartWorld = glm::inverse(projection * modelview) * rayStartNDC;
    glm::vec4 rayEndWorld = glm::inverse(projection * modelview) * rayEndNDC;

    rayStartWorld /= rayStartWorld.w;
    rayEndWorld /= rayEndWorld.w;

    start = glm::vec3(rayStartWorld);
    end = glm::vec3(rayEndWorld);
}
```

### Implementacion de la clase SphereSelector

```cpp
void SphereSelector::setup(SphereGrid* grid) {
    this->grid = grid;
    selectedSphere = nullptr;
}

void SphereSelector::mousePressed(int x, int y, ofEasyCam& cam) {
    if (!grid) return;

    glm::vec3 rayStart, rayEnd;
    RayCaster::mouseToRay(x, y, cam, rayStart, rayEnd);
    glm::vec3 rayDir = rayEnd - rayStart;

    // Deseleccionar todas primero
    for (auto& sphere : grid->getSpheres()) {
        sphere.setSelected(false);
    }

    // Buscar colisión
    selectedSphere = nullptr;
    for (auto& sphere : grid->getSpheres()) {
        glm::vec3 intersection;
        if (RayCaster::intersectSphere(rayStart, rayDir, sphere, intersection)) {
            sphere.setSelected(true);
            selectedSphere = &sphere;
            break;
        }
    }
}

const Sphere* SphereSelector::getSelectedSphere() const {
    return selectedSphere;
}
```

### Implementacion de la clase ofApp (aplicativo principal)
Mas formulas cortesia de **DeepSeek** porque todavia le tengo miedo a los numeros.
```cpp
void ofApp::setup() {
    ofSetVerticalSync(true);
    font.load("verdana.ttf", 12);

    grid.setup(ofGetWidth(), ofGetHeight());
    selector.setup(&grid);
}

void ofApp::update() {}

void ofApp::draw() {
    ofBackground(0);

    cam.begin();
    grid.draw();
    cam.end();

    
    if (const Sphere* selected = selector.getSelectedSphere()) { // Mostrar información de la esfera seleccionada
        string info = "Selected Sphere:\n";
        info += "Position: " + ofToString(selected->getPosition()) + "\n";
        info += "Radius: " + ofToString(selected->getRadius()) + "\n\n";
        info += "Grid Parameters:\n";
        info += "Step: " + ofToString(grid.getCurrentStep()) + " (i/d to change)\n";
        info += "Dist Div: " + ofToString(grid.getCurrentDistDiv()) + " (1/2 to change)\n";
        info += "Amplitude: " + ofToString(grid.getCurrentAmplitude()) + " (8/9 to change)";

        ofSetColor(255);
        font.drawString(info, 20, 60);
    }
}

void ofApp::keyPressed(int key) {
    int step = grid.getCurrentStep();
    float distDiv = grid.getCurrentDistDiv();
    float amplitude = grid.getCurrentAmplitude();

    switch (key) {
    case 'i': step += 5; break;
    case 'd': step = max(5, step - 5); break;
    case '1': distDiv += 5.0f; break;
    case '2': distDiv = max(1.0f, distDiv - 5.0f); break;
    case '8': amplitude += 10.0f; break;
    case '9': amplitude = max(0.0f, amplitude - 10.0f); break;
    default: return;
    }

    grid.generateGrid(step, distDiv, amplitude);
}

void ofApp::mousePressed(int x, int y, int button) {
    selector.mousePressed(x, y, cam);
}
```