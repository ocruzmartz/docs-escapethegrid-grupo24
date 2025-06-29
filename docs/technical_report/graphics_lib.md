# **Biblioteca Gráfica y Manual de Uso**

La creación de una aplicación gráfica interactiva en C++ requiere dos componentes fundamentales: una biblioteca que gestione los gráficos y un sistema que construya el proyecto. Para "Escape the Grid", se seleccionó la biblioteca **SFML** y se gestionó a través del sistema de construcción **CMake.**

## **Proceso de Investigación y Selección de la Biblioteca Gráfica: SFML**

Como parte de los requisitos del proyecto, se realizó una investigación de las alternativas disponibles para el desarrollo de interfaces gráficas (como Qt, ImGui, SDL2, Raylib, etc.) en C++. El objetivo era encontrar una herramienta que ofreciera un balance óptimo entre rendimiento, facilidad de uso, control y compatibilidad con un enfoque de desarrollo moderno orientado a objetos.

### **Alternativas Consideradas**

- **[SDL2](https://www.libsdl.org/):** La primera opción considerada fue SDL2, una biblioteca de bajo nivel muy robusta y multiplataforma, excelente para juegos. Su API es bastante completa y está basada en C. Integrarla en un proyecto C++ orientado a objetos como el nuestro habría requerido la creación de "wrappers" o clases envoltorio para mantener un diseño limpio, añadiendo una capa de complejidad adicional que podría haber complicado el desarrollo.

- **[ImGui](https://github.com/ocornut/imgui):** Antes de llegar a SFML, se consideró ImGui. Es una biblioteca de interfaz gráfica bastante completa, pero con un propósito diferente. Es una GUI de "modo inmediato" diseñada para crear herramientas de desarrollo y depuración sobre un motor gráfico existente. Como tal, no es una biblioteca de renderizado por sí misma, por lo que habríamos necesitado igualmente SFML o SDL2 para dibujar el juego principal.

### **Justificación de la Selección de SFML**

Tras evaluar las alternativas, se seleccionó SFML por ser la opción que ofrecía el mejor equilibrio para los objetivos del proyecto y por su facilidad de integración con CMake. Las razones específicas incluyen:

- **API Moderna y Orientada a Objetos:** SFML proporciona una interfaz de C++ nativa y limpia. Clases como `sf::RenderWindow`, `sf::RectangleShape` y `sf::Text` son intuitivas y permitieron un desarrollo rápido y un código más legible y mantenible.

- **Modularidad:** La biblioteca está dividida en módulos (System, Window, Graphics, Audio, Network). Esto permitió enlazar únicamente los componentes necesarios para nuestro proyecto (Graphics, Window, System), manteniendo las dependencias al mínimo, cumpliendo con el requisito de "Modularidad" del proyecto.

- **Rendimiento con Aceleración por Hardware:** SFML utiliza OpenGL de forma interna para el renderizado, lo que garantiza que todas las operaciones de dibujado sean eficientes y aceleradas por la GPU sin necesidad de gestionar directamente la complejidad de una API gráfica de bajo nivel.

- **Facilidad de Uso:** Es ideal para proyectos 2D, ya que tareas como cargar una fuente o dibujar un polígono se realizan con muy pocas líneas de código.

### **Integración con el Sistema de Construcción CMake**

**CMake** es una herramienta que automatiza el proceso de compilación. No compila el código directamente, sino que utiliza un archivo de configuración (`CMakeLists.txt`) para generar los archivos de proyecto nativos para un entorno específico (por ejemplo, una solución de Visual Studio en Windows o un Makefile en Linux).

Para asegurar este proceso de compilación robusto, multiplataforma y mantenible, es que se utilizó **CMake**, siguiendo la metodología recomendada por la propia documentación de SFML ([SFML with the CMake Project Template](https://www.sfml-dev.org/tutorials/3.0/getting-started/cmake/)). En lugar de enlazar manualmente las librerías, se aprovechó la capacidad de CMake para automatizar este proceso.

En la raíz del proyecto se encuentra el archivo `CMakeLists.txt` que contiene dos comandos cruciales para esta integración:

1. `find_package(SFML 3.0 REQUIRED COMPONENTS ...)`: Este comando le pide a CMake que busque en el sistema una instalación de SFML que cumpla con los requisitos. CMake se encarga de localizar las cabeceras (.hpp) y las librerías (.lib/.dll) necesarias.

2. `target_link_libraries(game PRIVATE SFML::...)`: Una vez que CMake ha encontrado SFML, este comando le dice al enlazador (linker) que conecte nuestro ejecutable (game) con los módulos de SFML correspondientes. CMake se encarga de resolver las rutas y dependencias correctas para el sistema operativo en el que se esté compilando.

Este enfoque automatiza la gestión de dependencias, haciendo que el proyecto sea fácil de compilar en diferentes máquinas y reduciendo la posibilidad de errores.

## **Manual de Uso de SFML**

SFML proporciona una API orientada a objetos que agiliza el desarrollo de aplicaciones multimedia. A continuación, se presentan los componentes fundamentales de la biblioteca, cada uno ilustrado con un ejemplo de código que demuestra su uso práctico:


- **`sf::RenderWindow` (Módulo `window`):** Es el pilar de cualquier aplicación SFML. Representa la ventana del sistema operativo y actúa como el lienzo sobre el cual se realizan todas las operaciones de dibujado. Gestiona el bucle principal del programa (`isOpen()`) y el ciclo de renderizado (`clear()`, `draw()`, `display()`).

    ```cpp
    // Crea una ventana de 800x600 píxeles con un título
    sf::RenderWindow window(sf::VideoMode({800, 600}), "Mi Ventana");
    while (window.isOpen()) {
        // ... Lógica de eventos y actualización ...
        window.clear(); // Limpia la pantalla
        // ... window.draw(...) para dibujar objetos ...
        window.display(); // Muestra el nuevo fotograma
    }
    ```

- **`sf::Event` (Módulo `window`):** Encapsula todos los eventos de entrada del usuario y del sistema (movimiento del ratón, clics, pulsaciones de teclado, cierre de la ventana, etc.). Los eventos se procesan en una cola que se consulta en cada fotograma mediante `pollEvent()`.

    ```cpp
    // Dentro del bucle principal (while window.isOpen())
    while (const std::optional<sf::Event> event = window.pollEvent()) {
        // El evento más común: cerrar la ventana
        if (event->is<sf::Event::Closed>()) {
            window.close();
        }
    }
    ```

- **`sf::Drawable` y `sf::Transformable` (Módulo `graphics`):** Son las clases base para cualquier objeto que se pueda dibujar y transformar (mover, rotar, escalar). Las clases más comunes que heredan de ellas son:
    - **`sf::Shape`:** Base para formas geométricas. `sf::ConvexShape` es una de sus derivadas, que permite crear polígonos convexos personalizados.
    - **`sf::Sprite`:** Para dibujar imágenes y texturas.
    - **`sf::Text`:** Para renderizar texto en pantalla.

    ```cpp
    // Crea una forma, la personaliza y la posiciona
    sf::CircleShape circulo(50.f); // Radio de 50 píxeles
    circulo.setFillColor(sf::Color::Green);
    circulo.setPosition({100.f, 100.f});
    // Para dibujarlo: window.draw(circulo);
    ```

- **`sf::Clock` (Módulo `system`):** Proporciona una forma sencilla y precisa de medir el tiempo transcurrido. Es esencial para implementar animaciones, físicas o cualquier lógica que dependa del tiempo y no de la velocidad de los fotogramas.

    ```cpp
    sf::Clock clock; // El cronómetro empieza a contar aquí
    // ... más tarde en el código ...
    sf::Time elapsed = clock.getElapsedTime();
    // Reinicia el cronómetro para la siguiente medición
    clock.restart();
    ```
### Demostración de Uso de SFML
A continuación, se presenta un ejemplo completo que utiliza varias funcionalidades de SFML, incluyendo la creación de una ventana, el manejo de eventos, el renderizado de gráficos y la reproducción de audio:

    ```cpp
    #include <SFML/Audio.hpp>
    #include <SFML/Graphics.hpp>
    
    int main()
    {
        // Create the main window
        sf::RenderWindow window(sf::VideoMode({800, 600}), "SFML window");
    
        // Load a sprite to display
        const sf::Texture texture("cute_image.jpg");
        sf::Sprite sprite(texture);
    
        // Create a graphical text to display
        const sf::Font font("arial.ttf");
        sf::Text text(font, "Hello SFML", 50);
    
        // Load a music to play
        sf::Music music("nice_music.ogg");
    
        // Play the music
        music.play();
    
        // Start the game loop
        while (window.isOpen())
        {
            // Process events
            while (const std::optional event = window.pollEvent())
            {
                // Close window: exit
                if (event->is<sf::Event::Closed>())
                    window.close();
            }
    
            // Clear screen
            window.clear();
    
            // Draw the sprite
            window.draw(sprite);
    
            // Draw the string
            window.draw(text);
    
            // Update the window
            window.display();
        }
    }
    ```
    *Ejemplo de uso de algunas de las funcionalidades de SFML extraído de la [documentación oficial](https://www.sfml-dev.org/documentation/3.0.0/).*


## **Uso de SFML en el Proyecto: Un Enfoque Práctico**

Una vez establecida la base de SFML con cada uno de sus módulos (en nuestro caso, `window`, `graphics` y `system`) y comprendidas sus características, se aplicaron en el desarrollo del juego "Escape the Grid".

- **Gestión de la Ventana y Bucle Principal (`sf::RenderWindow`)**
  La clase `sf::RenderWindow` es el pilar de la aplicación. Encapsula la ventana del sistema operativo y es el objetivo de todas las operaciones de dibujado. El bucle principal del juego se estructura en torno al método `window.isOpen()` por el cual, la clase, controla el ciclo de vida de la aplicación. Dentro de este bucle, se sigue el ciclo de renderizado en tiempo real basado en doble búfer para evitar parpadeos (flickering):

      - **Procesamiento de Eventos:** Se gestiona la entrada del usuario.
      - **Actualización de Lógica:** Se actualiza el estado de todos los objetos del juego.
      - **Renderizado:** Se limpia el fotograma anterior (`window.clear()`), se dibujan todos los objetos en un búfer oculto (`window.draw(...)`), y finalmente se muestra el resultado en pantalla (`window.display()`). Además, se usó `setFramerateLimit(60)` para estabilizar la velocidad de actualización.


          ```cpp
          // En main.cpp
          sf::RenderWindow window(sf::VideoMode({1280, 720}), "Escape the Grid");
          window.setFramerateLimit(60);

          while (window.isOpen())
          {
          // ... Lógica de eventos y actualización ...
              window.clear(sf::Color(44, 62, 80));
              // ... Llamadas a window.draw(...) para cada objeto ...
              window.display();
          }
          ```

- **Sistema de Eventos para la Interacción (`sf::Event`)**
  Para cumplir con el requisito de una interacción 100% por ratón, se utilizó el sistema de eventos de SFML. En cada fotograma, un bucle while (`window.pollEvent(event)`) procesa la cola de eventos del sistema operativo. Se optó por un enfoque moderno y seguro de C++ 17, `event->getIf<T>()`, para gestionar los diferentes tipos de eventos, enfocándonos en `sf::Event::MouseButtonPressed`:

    ```cpp
    // En main.cpp, dentro del bucle de eventos
    while (const std::optional<sf::Event> event = window.pollEvent())
    {
        if (const auto *mousePress = event->getIf<sf::Event::MouseButtonPressed>()) {
            if (mousePress->button == sf::Mouse::Button::Left) {
                    // Lógica para clics del ratón...
                    sf::Vector2f mousePos = window.mapPixelToCoords(mousePress->position);
                    // ...
            }
        }
    }
    ```

- **Renderizado de Entidades Gráficas (`sf::ConvexShape`)**
  El requisito de una malla de triángulos exigía una solución de renderizado flexible (polígonos convexos). La clase `sf::ConvexShape` fue la herramienta ideal. A diferencia de formas predefinidas, permite crear cualquier polígono convexo definiendo la posición de cada uno de sus vértices. Para cada `TrianguloNode` en nuestro `GridModel`, se creó una instancia de `sf::ConvexShape`y se le asignan 3 vértices para formar el triángulo exacto en su posición, dándonos control total sobre la geometría del mapa.

    ```cpp
    // Concepto de implementación en GridModel.cpp
    TrianguloNode nuevoNodo;
    nuevoNodo.shape.setPointCount(3);
    nuevoNodo.shape.setPoint(0, {x, y});
    nuevoNodo.shape.setPoint(1, {x + lado, y});
    nuevoNodo.shape.setPoint(2, {x + anchoMedio, y + altura});
    ```

-  **Control del Tiempo y Animaciones (`sf::Clock`)**
Para crear una experiencia de usuario fluida, se implementaron múltiples animaciones. La clase `sf::Clock` fue la base para toda la temporización. Actúa como un cronómetro de alta precisión que se puede reiniciar y consultar. El patrón utilizado para todas las animaciones fue:

    - Iniciar un reloj (`restart()`) al comenzar una animación.
    - En cada fotograma, calcular el progreso (un valor de 0.0 a 1.0) dividiendo el tiempo transcurrido por la duración total de la animación.
    - Usar este progreso para interpolar una propiedad (posición, color, etc.).

    ```cpp
    // En main.cpp, dentro del bucle principal
    if (isPlayerMoving) {
        float progress = playerMoveClock.getElapsedTime().asSeconds() / MOVE_DURATION_SECONDS;
        if (progress >= 1.0f) {
            isPlayerMoving = false;
            jugador.setPosicion(playerTargetPos); // Asegura la posición final
        } else {
            // Interpola la posición entre el inicio y el destino
            sf::Vector2f newPos = playerStartPos + (playerTargetPos - playerStartPos) * progress;
            jugador.setPosicion(newPos);
        }
    }
    ```
!!!note
    Para más información sobre **SFML** y su uso con **CMake**, se puede consultar [SFML with the CMake Project Template](https://www.sfml-dev.org/tutorials/3.0/getting-started/cmake/) y su [documentación](https://www.sfml-dev.org/documentation/3.0.0/).