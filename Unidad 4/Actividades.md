# Actividad 1

## 1. Entendiendo la aplicación

Después de analizar el código con ChatGPT y el depurador de Visual Studio, entendí que este programa crea una "serpiente" interactiva compuesta por nodos conectados (lista enlazada) que sigue el cursor del mouse. Usando el depurador, pude ver cómo:

- Al inicio (`setup()`), se crean 10 nodos en la posición central de la pantalla
- En cada frame (`update()`), la posición de cada nodo se actualiza para seguir al nodo anterior
- El primer nodo sigue directamente al cursor del mouse
- Al presionar 'c' (`keyPressed()`), se limpia toda la lista

## 2. Evaluaciones formativas

ChatGPT me hizo varias preguntas para verificar mi comprensión:

**Pregunta:** ¿Qué sucede cuando se llama a `addNode()`?
**Respuesta:** Se crea un nuevo nodo con las coordenadas dadas y se añade al final de la lista (tail). Verifiqué en el depurador viendo cómo `tail` apunta al nuevo nodo y `size` aumenta.

**Pregunta:** ¿Cómo se actualizan las posiciones en `update()`?
**Respuesta:** Cada nodo toma la posición del nodo anterior, empezando por el head que toma la posición del mouse. Confirmé esto con breakpoints viendo cómo las coordenadas se propagan por la lista.

## 3. Lista enlazada vs arreglo

Una lista enlazada es una estructura donde cada elemento (nodo) contiene:
- Sus datos (en este caso, coordenadas x,y)
- Un puntero al siguiente nodo

Diferencias con arreglos:
- **Memoria no contigua**: Los nodos pueden estar en cualquier lugar de la memoria
- **Tamaño dinámico**: Puede crecer/shrink fácilmente
- **Inserción/eliminación**: Más eficiente para operaciones en medio de la lista

## 4. Vinculación de nodos

Los nodos se vinculan mediante el puntero `next` en la clase `Node`. Cada nodo contiene:
```cpp
float x, y;       // Datos
Node* next;       // Puntero al siguiente nodo
```

La lista mantiene referencia al primer (`head`) y último nodo (`tail`).

## 5. Gestión de memoria

- **Creación**: Usa `new` para asignar memoria dinámica:
  ```cpp
  Node* newNode = new Node(x, y);
  ```
- **Destrucción**: Usa `delete` para liberar memoria:
  ```cpp
  delete current;
  ```

## 6. Ventajas sobre arreglos

Para inserciones/eliminaciones en posiciones intermedias:
- **Lista enlazada**: Solo necesita cambiar algunos punteros (O(1) si conoces la posición)
- **Arreglo**: Necesita desplazar elementos (O(n))

## 7. Prevención de fugas de memoria

El programa asegura la liberación de memoria mediante:
1. Destructor `~LinkedList()` que llama a `clear()`
2. Método `clear()` que recorre toda la lista liberando cada nodo
3. Llamada explícita a `clear()` al presionar 'c'

## 8. Proceso de `clear()`

1. Comienza en `head`
2. Guarda referencia al siguiente nodo (`nextNode`)
3. Libera el nodo actual con `delete`
4. Avanza al siguiente nodo
5. Repite hasta llegar al final (nullptr)
6. Finalmente establece `head` y `tail` a nullptr

## 9. Añadir nodos al final

Al agregar un nuevo nodo:
1. Se crea un nuevo nodo
2. Se actualiza `tail->next` para que apunte al nuevo nodo
3. Se actualiza `tail` para que sea el nuevo nodo
4. Se incrementa `size`

**Rendimiento**: O(1) porque tenemos referencia directa al tail.

## 10. Caso de uso favorable

**Escenario**: Un editor de gráficos vectoriales donde los usuarios pueden añadir/eliminar puntos libremente.

**Ventajas**:
- Inserción/eliminación rápida de puntos intermedios
- No necesita realocación como los arreglos dinámicos
- Tamaño se ajusta automáticamente

## 11. Diseño de estructura personalizada

Para una aplicación creativa consideraría:
1. **Responsabilidad única**: Cada nodo solo debe manejar sus datos
2. **Ciclo de vida claro**: Definir bien cuándo se crean/destruyen nodos
3. **Iteración eficiente**: Métodos para recorrer la lista
4. **RAII**: Usar constructores/destructores para gestión automática

## 12. C++ vs C# (gestión de memoria)

**C++**:
- Ventajas: Control total, eficiencia, determinismo
- Desafíos: Mayor complejidad, riesgo de leaks/dangling pointers

**C#**:
- Ventajas: Automático, más seguro
- Desafíos: Menos control, overhead por GC

## 13. Optimización para arte generativo

Para optimizar:
1. **Pool de nodos**: Reutilizar nodos en lugar de crear/eliminar constantemente
2. **Actualización por lotes**: Procesar múltiples nodos eficientemente
3. **Monitoreo**: Herramientas para detectar leaks
4. **Smart pointers**: Considerar usar `unique_ptr` para gestión automática

## 14. Pruebas realizadas

ChatGPT sugirió estas pruebas que implementé:

1. **Prueba de creación**:
```cpp
// Verificar que se crean exactamente 10 nodos
assert(snake.size == 10);
```

2. **Prueba de movimiento**:
```cpp
// Mover mouse y verificar que head sigue al cursor
snake.update(100, 100);
assert(snake.head->x == 100 && snake.head->y == 100);
```

3. **Prueba de limpieza**:
```cpp
snake.clear();
assert(snake.size == 0 && snake.head == nullptr);
```

Los resultados coincidieron con lo esperado, confirmando el correcto funcionamiento.

# Actividad 2

## 1. Entendiendo las aplicaciones

### Para la Pila (Stack):
Después de analizar el código con ChatGPT y el depurador, entendí que:
- La pila sigue el principio LIFO (Last In, First Out)
- Cada nodo almacena una posición (x,y) y un puntero al siguiente nodo
- Al presionar 'a' se apila un nuevo círculo en la posición del mouse
- Al presionar 'd' se desapila el último círculo añadido

### Para la Cola (Queue):
El análisis mostró que:
- La cola sigue el principio FIFO (First In, First Out)
- Usa dos punteros: front (frente) y rear (final)
- 'a' encola un nuevo círculo al final
- 'd' desencola el círculo del frente

## 2. Evaluaciones formativas

ChatGPT generó estas preguntas y verifiqué las respuestas con el depurador:

**Pregunta (Stack):** ¿Qué sucede en memoria al apilar?
**Respuesta verificada:** Se crea un nuevo nodo con `new`, su `next` apunta al actual `top`, y `top` se actualiza al nuevo nodo.

**Pregunta (Queue):** ¿Cómo se manejan front y rear al desencolar?
**Respuesta verificada:** `front` avanza al siguiente nodo, y si queda vacía, `rear` también se pone a nullptr.

## 3. Pruebas realizadas

### Para la Pila:
1. **Prueba de apilado**:
```cpp
for(int i=0; i<5; i++) circleStack.push(i*10, i*10);
assert(circleStack.top != nullptr);
```

2. **Prueba de desapilado**:
```cpp
circleStack.pop();
assert(circleStack.top->position.x == 30); // Verifica el nuevo top
```

### Para la Cola:
1. **Prueba de encolado**:
```cpp
circleQueue.enqueue(100,100);
assert(circleQueue.rear->position.x == 100);
```

2. **Prueba de desencolado**:
```cpp
circleQueue.enqueue(200,200);
circleQueue.dequeue();
assert(circleQueue.front->position.x == 200);
```

## Reflexiones sobre el Stack

1. **Gestión de memoria manual**:
   - `new` asigna memoria en el heap para cada nodo
   - `delete` en `pop()` y `clear()` libera esta memoria
   - Ventaja: Control preciso
   - Desventaja: Mayor riesgo de leaks si no se maneja bien

2. **Importancia de liberar memoria**:
   - Sin `delete`, cada nodo desapilado causaría un leak
   - En aplicaciones de arte generativo que corren horas, esto podría consumir toda la memoria

3. **STL vs Manual**:
   - `std::stack` maneja memoria automáticamente
   - Implementación manual permite:
     - Control exacto sobre la representación visual
     - Integración directa con OpenFrameworks

4. **Estructura LIFO**:
   - Útil para algoritmos como:
     - Deshacer/rehacer en editores
     - Recorrido de árboles
     - Análisis sintáctico

5. **Tipos complejos**:
   - Para objetos complejos, necesitaríamos:
     - Implementar destructores adecuados
     - Posiblemente usar smart pointers
     - Ejemplo:
       ```cpp
       class ComplexNode {
           ofImage texture;
           ofVec3f position;
           // Destructor que libera recursos
           ~ComplexNode() { texture.clear(); }
       };
       ```

## Autoevaluación Stack

1. **Sí**, puedo explicar el proceso completo incluyendo cómo `new` y `delete` gestionan memoria.

2. **Sí**, identifico leaks cuando falta `delete` y puedo corregirlos.

3. **Sí**, añadiría una función de búsqueda que recorra la pila sin modificar `top`.

4. **Sí**, entiendo LIFO. Ejemplo: sistema de deshacer operaciones.

5. **Sí**, para tipos complejos usaría RAII y manejaría cuidadosamente recursos.

## Reflexiones sobre la Queue

1. **Gestión de memoria**:
   - Similar al stack pero con dos punteros
   - `enqueue` añade al `rear`
   - `dequeue` elimina del `front`

2. **Desafíos vs Stack**:
   - Mayor complejidad al manejar dos punteros
   - Caso especial cuando queue queda vacía
   - Ejemplo:
     ```cpp
     if(front == nullptr) {
         rear = nullptr; // Importante!
     }
     ```

3. **Estructura FIFO**:
   - Ideal para:
     - Buffers de animación
     - Sistemas de partículas
     - Manejo de eventos

4. **Queue circular**:
   - Ventaja: Reutiliza memoria
   - Implementación:
     ```cpp
     rear->next = front; // Cierra el círculo
     ```

5. **Problemas con punteros**:
   - Pérdida de nodos si no se actualizan bien front/rear
   - Solución: Verificar siempre nullptr al desencolar

## Autoevaluación Queue

1. **Sí**, puedo explicar encolado/desencolado y el rol de front/rear.

2. **Sí**, identifico cuando front/rear no se sincronizan correctamente.

3. **Sí**, implementaría una queue circular redefiniendo las operaciones.

4. **Sí**, entiendo FIFO. Ejemplo: sistema de partículas que aparecen y desaparecen en orden.

5. **Sí**, para datos complejos aseguraría liberación adecuada en `dequeue()`.

## Conclusiones

1. **Gestión de memoria** en C++ ofrece control pero requiere disciplina
2. **Pilas** son ideales para operaciones inversas (LIFO)
3. **Colas** son mejores para procesamiento en orden (FIFO)
4. **Arte generativo** se beneficia de estas estructuras para:
   - Manejar historial de estados
   - Controlar flujos de partículas
   - Gestionar elementos visuales dinámicos
