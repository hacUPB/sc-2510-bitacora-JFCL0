# Actividad 1

Aquí tienes las respuestas claras y técnicas a tus preguntas sobre gráficos por computadora:

### 1. ¿Qué son los vértices?
Los vértices (vertices) son puntos en el espacio 3D (con coordenadas x, y, z) que definen la estructura geométrica de un modelo. Son los "puntos de conexión" que forman las aristas de los polígonos (generalmente triángulos) en una malla 3D.

### 2. ¿Con qué figura geométrica se dibuja en 3D?
El elemento fundamental es el **triángulo** (primitiva más simple y garantiza planitud). También se usan:
- Quads (2 triángulos unidos)
- Strips/Fans (optimización de triángulos conectados)
- Patches (para teselación)

### 3. ¿Qué es un shader?
Programa que corre en la GPU para controlar etapas específicas del pipeline de renderizado. Los principales tipos son:
- Vertex Shader (procesa vértices)
- Fragment Shader (procesa píxeles)
- Geometry Shader (manipula primitivas)
- Compute Shader (propósitos generales)

### 4. ¿Cómo se le llaman a los grupos de píxeles de un mismo triángulo?
**Fragmentos** (fragments). Cada fragmento contiene:
- Posición en pantalla
- Profundidad (z-buffer)
- Datos interpolados (color, normales, UVs)

### 5. ¿Qué es un fragment shader?
Programa que determina el color final de cada fragmento (píxel potencial). Maneja:
- Iluminación
- Texturas
- Efectos (transparencia, reflejos)

### 6. ¿Qué es un vertex shader?
Programa que transforma cada vértice de:
1. Coordenadas locales → Espacio mundo → Espacio cámara → Coordenadas de pantalla
2. Puede modificar posición, normales, UVs

### 7. ¿Al proceso de determinar qué píxeles del display va a cubrir cada triángulo de una mesh se le llama?
**Rasterización** (o scan conversion). Convierte primitivas vectoriales en píxeles.

### 8. ¿Qué es el render pipeline?
Secuencia de etapas para convertir modelos 3D en una imagen 2D:
```
1. Input Assembler → 2. Vertex Shader → 3. Tessellation → 4. Geometry Shader → 5. Rasterization → 6. Fragment Shader → 7. Output Merger
```

### 9. ¿Hay diferencia entre aplicar color o textura?
Sí:
- **Color único**: Valor RGB constante en toda la superficie
- **Textura**: Mapeo de imagen (con coordenadas UV) que permite variación espacial de color/detalle

### 10. ¿Diferencia entre textura y material?
- **Textura**: Imagen 2D/3D que almacena datos (color, normales, roughness)
- **Material**: Conjunto de propiedades + shaders + texturas que definen la apariencia superficial (ej: metal vs plástico)

### 11. ¿Transformaciones para mover un vértice a pantalla?
Pipeline de transformación:
```
1. Model Matrix (Local → World)
2. View Matrix (World → Camera)
3. Projection Matrix (Camera → Clip Space)
4. Perspective Divide (Clip → NDC)
5. Viewport Transform (NDC → Screen)
```

### 12. ¿Proceso de convertir triángulos en fragmentos?
**Rasterización** (como en pregunta 7). Incluye:
- Determinación de cobertura de píxeles
- Interpolación de atributos (Gouraud shading)

### 13. ¿Qué es el framebuffer?
Memoria de la GPU que almacena los píxeles finales para mostrar, compuesto por:
- Color buffer (imagen visible)
- Depth buffer (profundidad por píxel)
- Stencil buffer (máscaras)

### 14. ¿Para qué sirve el Z-buffer?
**Occlusión correcta**: Almacena la profundidad (coordenada Z) de cada fragmento para:
- Resolver qué objetos están delante/atrás (depth testing)
- Evitar dibujar fragmentos ocultos (optimización)


# Actividad 2

Comienza realizando la lectura de la introducción del tutorial [Introducing Shaders](https://openframeworks.cc/ofBook/chapters/shaders.html). Realiza la sección Your First Shader, pero antes de ejecutar el código, realiza un pequeño experimento. Modifica ligeramente el método draw:

```cpp
void ofApp::draw(){
    ofSetColor(255);

    //shader.begin();

    ofDrawRectangle(0, 0, ofGetWidth(), ofGetHeight());

    //shader.end();
}
```

Observa la salida.

Ahora ejecuta el código original. ¿Qué resultados obtuviste?

- ¿Cómo funciona?  
Primero, en `main.cpp` se indica al programa que se va a estar utilizando el shader **GLV**, tras esto, se debe acceder a la carpeta `bin` del proyecto y agregar una subcarpeta `data`, en la cual se crea una carpeta `shadersGL3`. Dentro de esta carpeta se crean 2 archivos, `shader.vert`, que sera el **Vertex Shader** y el archivo `shader.frag`, que sera el **Fragment Shader**. Tras esto se agrega la linea `ofShader shader` en el archivo `ofApp.h`, que permite ejecutar las funciones necesarias para ejecutar el shader. Finalmente, en el archivo `ofApp.cpp` se modifican los metodos `ofApp::setup()`, donde se **inicializan los shaders de ambos archivos en la carpeta data**. Tras esto, en el mismo archivo se modifica el metodo `ofApp::draw()`, en el cual antes de dar la orden de dibujar se utiliza **el metodo que ejecuta los shaders** `shader.begin()`, el cual se da de la siguiente manera: Primero, sabiendo la posicion y tamaño que se ordeno para la figura, envia la posicion de los **vectores** requeridos para construir la figura en la forma de `vec4 position`, el `Vertex Shader` recibe esta informacion y ejecuta una unica operacion `gl_Position = modelViewProjectionMatrix * position`. Esta operacion hace que los procesos requeridos para determinar la posicion de un objeto a partir de sus vertices, **pasando de mover un modelo de un Model Space a un World Space, luego calcula el cambio de World Space a Camera Space y finalmente, de Camera Space a View Screen** mediante la realizacion de operaciones matriciales. Cuando finaliza esta operacion, el Vertex Shader envia la informacion de la ubicacion de los pixeles al **Fragment Shader**, que lo que hace es, a partir de las ordenes en el, calcula que color debe tener **cada pixel en pantalla basado en la informacion recibida desde el Vertex Shader** y hace envio de esta informacion a la GPU. Volviendo a `ofApp.cpp`, tras dar la orden de dibujar un rectangulo **mientras** el proceso de los shaders esta activo, finalmente se finaliza el proceso de los calculos de los shaders mediante la linea `shader.end()`.
- ¿Qué resultados obtuviste?  
Un difuminado generado a partir de una operacion en el interior del `Fragment Shader`, que indica que el valor r de color ira incrementando a medida que el pixel tenga un valor de posicion en x mas alto, y lo mismo con el valor f de color en torno al eje y.
- ¿Estás usando un vertex shader?  
Si.
- ¿Estás usando un fragment shader?  
Si.
- Analiza el código de los shaders. ¿Qué hace cada uno?  
Codigo del Vertex Shader: Este se encarga de transformar la posicion del objeto desde un espacio 3D (incluso si el programa no lo considera asi) a posiciones de vertices como se verian en pantalla mediante procesarlas bajo una serie de operaciones matriciales. Su output es un tipo de concepto que el Fragment Shader puede entender.

    ```cpp
    // vertex shader

    #version 150

    uniform mat4 modelViewProjectionMatrix;
    in vec4 position;

    void main(){
        gl_Position = modelViewProjectionMatrix * position;
    }
    ```

    Codigo del Fragment Shader: Este primero genera el espacio que ocupara la ventana, tras esto establece las reglas (a mayor sea la posicion en x, mayor sera el valor en r; a mayor sea el valor en y, mayor sera el valor en g, b = 1, a = 1) y envia a la GPU el color que tendra asignado cada pixel.

    ```cpp
    // fragment shader

    #version 150

    out vec4 outputColor;

    void main()
    {
        // gl_FragCoord contains the window relative coordinate for the fragment.
        // we use gl_FragCoord.x position to control the red color value.
        // we use gl_FragCoord.y position to control the green color value.
        // please note that all r, g, b, a values are between 0 and 1.

        float windowWidth = 1024.0;
        float windowHeight = 768.0;

        float r = gl_FragCoord.x / windowWidth;
        float g = gl_FragCoord.y / windowHeight;
        float b = 1.0;
        float a = 1.0;
        outputColor = vec4(r, g, b, a);
    }
    ```

# Actividad 3

Ahora vas a pasar información personalizada de tu programa a los shaders. Vas a leer con detenimiento el tutorial Adding Uniforms.

- ¿Qué es un uniform?  
Un uniform es una forma de enviar variables desde el programa principal hasta los shaders. Estas variables son usadas en diferentes operaciones que el programa pueda requerir.
- ¿Cómo funciona el código de aplicación, los shaders y cómo se comunican estos?  
El codigo de la aplicacion primero genera un plano. En el `void draw()` primero se crea un `float percentX` que sera el encargado de cambiar el color segun el mouse se mueve alrededor del eje x. Justo tras esto se llama el primer **Uniform**, siendo este `globalColor`, declarado en `shader.frag` con la linea de codigo `uniform vec4 globalColor;`, este **debe poder actualizarse segun el mouse se mueve en x**, esto se logra a partir de, primero, generar 2 colores, `colorRight` y `colorLeft`. Tras esto se genera una nueva variable `colorMix`, que a partir de una funcion `getLerped` genera un modificador a `colorLeft`, agregando valor **alpha** a `colorRight` a mayor sea el valor de `percentX`, sobreponiendo los 2 colores. Tan pronto esta lista la variable `colorMix`, su valor es asignado a `ofSetColor`, que es la forma de acceso que se tiene al `globalColor`.  
Tras esto aparece el segundo y ultimo **uniform**, evidente en la siguiente linea: `shader.setUniform1f("time", ofGetElapsedTimef)`. Esta linea es utilizada en `shader.vert` para generar una animacion. Esto se logra mediante primero crear un `float displacementHeight = 100`, el cual se usaria en la siguiente formula: `float displacementY = sin(time + (position.x / 100)) * displacementHeight`. Es en esta formula en la que se utiliza el valor del uniform **time**. 

Modifica el código de la actividad para cambiar el color de cada uno de los píxeles de la pantalla personalizando el fragment shader.

```cpp
#version 150
uniform vec4 globalColor;
out vec4 outputColor;

void main()
{
    float r = 1.0;
    float g = 5.0;
    float b = 0.5;
    float a = 1.0;
    outputColor = globalColor * vec4(r, g, b, a);
}
```

# Actividad 4

Vas a realizar la última actividad de esta experiencia de aprendizaje. Yo sé que quieres seguir haciendo más, pero tenemos un tiempo muy limitado.

Analiza el ejemplo Adding some interactivity.

- ¿Qué hace el código del ejemplo?  
Este codigo busca que en un rango alrededor del mouse se genere un efecto de repulsion a los vectores del plano.
- ¿Cómo funciona el código de aplicación, los shaders y cómo se comunican estos?  
En este codigo al momento de ejecutarse `void draw()` se genera un total de **3 Uniforms**, que se pueden agrupar en 2 funcionalidades, **para el Vertex** y **para el Fragment**.  
En el caso del Vertex este recibe **2 uniforms**, el primero se obtiene tras realizar las siguientes formulas: `float cx/cx = (ofGetWidth()/ofGetHeight())/2`, tras esto se realiza `float mx/my = (mouseX/mouseY) - (cx/cy)`. Teniendo estos valores se envia un primer **Uniform** `shader.setUniform1f("mouseRange", 150)`, e inmediatamente despues se envia un segundo **Uniform** `shader.setUniform2f("mousePos", mx, my)`. Estos dos son utilizados en el archivo `shader.vert`, en donde se realizan las respectivas operaciones para lograr que la posicion de los vectores sean repelidos por el mouse, haciendo uso de esas 2 variables para poder ejecutarse.  
En el caso del Fragment, ese recibe **un unico uniform** siendo este `mouseColor`,  que se define de una forma casi identica a la del ejercicio anterior, teniendo este un unico cambio, que ahora se debe enviar un vector **ya existente**, lo cual se logra utilizando la siguiente linea de codigo: `shader.setUniform4fv("mouseColor", &mouseColor[0])`, esto se logra ya que al enviar un array ya existente por **Uniform** este debe señalarse con un puntero a la direccion de su primer elemento. Este Uniform es utilizado mediante la linea `outputColor = mouseColor`.
- Realiza modificaciones a ofApp.cpp y al vertex shader para conseguir otros comportamientos.  

```cpp
//------------------------------------------------------------------------
// shader.vert
//------------------------------------------------------------------------
#version 150


uniform mat4 modelViewProjectionMatrix;
in vec4 position;

uniform float mouseRange;
uniform vec2 mousePos;
uniform vec4 mouseColor;

void main()
{
    
    vec4 pos = position;
    vec2 dir = pos.xy + mousePos; //cambio de - a +, vectores intentan acercarse
    float dist =  distance(pos.xy, mousePos);
    if(dist > 0.0 && dist < mouseRange) {

        float distNorm = dist / mouseRange;
        distNorm = 1.0 - distNorm;
        dir *= distNorm;
        pos.x += dir.x;
        pos.y += dir.y;
    }

    gl_Position = modelViewProjectionMatrix * pos;
}
//------------------------------------------------------------------------
// ofApp.cpp
//------------------------------------------------------------------------
void ofApp::draw() {
    shader.begin();
    float cx = ofGetWidth() / 2.0;
    float cy = ofGetHeight() / 2.0;
    float mx = mouseX - cx/2; //agregar /2 despues de cx, programa acerca los vectores a un lugar extraño
    float my = mouseY - cy/2; //agregar /2 despues de cy, programa acerca los vectores a un lugar extraño
    shader.setUniform1f("mouseRange", 150); 
    shader.setUniform2f("mousePos", mx, my);  

    float percentX = mouseX / (float)ofGetWidth();
    percentX = ofClamp(percentX, 0, 1);
    ofFloatColor colorLeft = ofColor::magenta;
    ofFloatColor colorRight = ofColor::blue;
    ofFloatColor colorMix = colorLeft.getLerped(colorRight, percentX);
    float mouseColor[4] = { colorMix.r, colorMix.g, colorMix.b, colorMix.a };
    shader.setUniform4fv("mouseColor", &mouseColor[0]); 

    ofTranslate(cx, cy);

    plane.drawWireframe();

    shader.end();
}
```
- Realiza modificaciones al fragment shader para conseguir otros comportamientos.  
```cpp
// fragment shader

#version 150

out vec4 outputColor;
uniform vec4 mouseColor;

void main()
{
    outputColor = mouseColor/2; //agregar /2 despues de mouseColor, color mas opaco
}
```

