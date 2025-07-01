# **Motor de Visualización y Renderizado**

El motor de visualización de "Escape the Grid" es el componente responsable de traducir el estado abstracto del juego (la lógica en `GridModel`, `GameState` y `Player`) en una representación gráfica interactiva para el usuario. Construido mediante las funciones de la biblioteca SFML, este motor no solo se encarga de "dibujar" objetos, sino también de proporcionar feedback visual dinámico y animaciones que mejoran la experiencia de juego.

La visualización se puede descomponer en los siguientes componentes clave:

## Representación del Entorno y Estado (`GridModel`)

La clase [`GridModel`](src/GridModel.hpp) es el núcleo de la visualización del mapa del juego. Para que el jugador pudiera entender el estado del juego de un solo vistazo, se utilizaron varias técnicas de retroalimentación visual directa.

- **Interpretación del Mapa desde Archivo:** La estructura del nivel se carga desde un archivo de texto (`.txt`). El método `cargarDesdeArchivo` interpreta cada carácter del archivo y lo asocia a un tipo de celda (`TipoCelda`), que a su vez define su color y comportamiento.

  ```text
  // Ejemplo de un archivo de mapa simple (mapa.txt)
  WWWWWWWW
  WS.1.T.F
  W..W...W
  W..2.3.W
  WWTT..WW
  ```
Donde:

- **`S`**: Punto de inicio del jugador.
- **`.`**: Celda de camino normal, transitable.
- **`W`**: Muro, celda no transitable.
- **`1`, `2`, `3`**: Ítems de secuencia de la "Caja Mágica", que deben ser recogidos en orden.
- **`T`**: Muro temporizado, que se convierte en muro (`W`) después de un número de turnos.
- **`F`**: Celda final u objetivo.

La correspondencia de caracteres se gestiona con un `switch` dentro de [`GridModel.cpp`](src/GridModel.cpp):

  ```cpp
  // En GridModel.cpp
          else
          {
              char caracter = mapaDeCaracteres[fila][col];
              switch (caracter)
              {
              case 'S':
                  tipo = TipoCelda::INICIO;
                  colorCelda = sf::Color(46, 204, 113);
                  break;
              case 'W':
                  tipo = TipoCelda::PARED;
                  colorCelda = sf::Color(52, 73, 94);
                  break;
              case 'F':
                  tipo = TipoCelda::FIN_OCULTO;
                  colorCelda = sf::Color(189, 195, 199);
                  posicionSalida = {(int)col, (int)fila};
                  break;
              case 'T':
                  tipo = TipoCelda::TIMED_WALL;
                  colorCelda = sf::Color(52, 152, 219);
                  break;
              case '1':
              case '2':
              case '3':
                  tipo = TipoCelda::ITEM_SECUENCIA;
                  colorCelda = sf::Color(241, 196, 15);
                  break;
              default:
                  tipo = TipoCelda::NORMAL;
                  colorCelda = sf::Color(189, 195, 199);
                  break;
              }
          }
  // ...
  ```

- **Generación Procedural de la Malla:** Durante la carga, el `GridModel` no solo lee el mapa, sino que calcula dinámicamente la geometría de cada triángulo. Basándose en el tamaño de la ventana y las dimensiones del mapa, determina el tamaño y la posición de cada `sf::ConvexShape` para que la malla ocupe el espacio de forma óptima.

- **Retroalimentación de Celdas Visitadas:** Para ayudar al jugador a orientarse, las celdas ya visitadas se marcan visualmente. La función `marcarNodoVisitado` toma el color actual del triángulo y lo oscurece, dejando un "rastro" visual claro.

  ```cpp
  // En GridModel.cpp
  void GridModel::marcarNodoVisitado(int nodeId)
  {
      if (nodeId < 0 || static_cast<size_t>(nodeId) >= nodosDelGrid.size())
          return;
      auto &nodo = nodosDelGrid[nodeId];
      if ((nodo.tipo == TipoCelda::NORMAL || nodo.tipo == TipoCelda::FIN_OCULTO || nodo.tipo == TipoCelda::ITEM_SECUENCIA) && !nodo.haSidoVisitado)
      {
          nodo.haSidoVisitado = true;
          sf::Color colorActual = nodo.shape.getFillColor();
          nodo.shape.setFillColor(sf::Color(colorActual.r / 1.5, colorActual.g / 1.5, colorActual.b / 1.5));
      }
  }
  // ...
  ```

- **Resaltado de Interacción ("Hover"):** Para mejorar la interactividad, la función `GridModel::actualizar` detecta en cada fotograma la posición del ratón. Si el cursor está sobre un triángulo válido, el borde de ese triángulo cambia a color blanco, dando una señal inequívoca de la selección.

  ```cpp
  // En GridModel.cpp
      for (size_t i = 0; i < nodosDelGrid.size(); ++i)
      {
          if (nodosDelGrid[i].tipo != TipoCelda::PARED && nodosDelGrid[i].tipo != TipoCelda::VACIO && nodosDelGrid[i].shape.getGlobalBounds().contains(posicionRaton))
          {
              nodosDelGrid[i].shape.setOutlineColor(sf::Color::White);
              idTrianguloResaltado = i;
              break;
          }
      }
  // ...
  ```

- **Dibujado del Grid:** El método `dibujar(sf::RenderWindow& ventana)` simplemente itera sobre el vector `nodosDelGrid` y llama a `ventana.draw()` para cada `sf::ConvexShape`, renderizando eficientemente todo el mapa.

## Visualización de Entidades y HUD

Sobre el mapa base se renderizan las entidades dinámicas y la interfaz de usuario (HUD).

- **El Jugador (`Player`):** La clase [`Player`](src/player.hpp) encapsula una `sf::CircleShape`. Su posición no es gestionada por la clase misma, sino por la lógica central en [`main.cpp`](src/main.cpp), que le asigna una nueva coordenada basándose en el nodo del grid en el que se encuentra.

- **Interfaz de Usuario (HUD):**  
  Actualmente, la ventana del juego se divide en dos paneles principales:
    - **Panel lateral de interfaz:**  
     Ubicado a la derecha, contiene toda la información relevante para el jugador (puntuación, turnos, objetivo actual) y los botones de interacción como "Reiniciar" y "Resolver". Este panel se implementa con un fondo (`sf::RectangleShape`) y varios objetos `sf::Text` para los textos y botones, todos actualizados dinámicamente según el estado del juego.

        ```cpp
        // En main.cpp
        window.draw(uiPanelBackground);
        window.draw(scoreText);
        window.draw(turnText);
        window.draw(sequenceText);
        window.draw(restartButtonShape);
        window.draw(restartButtonText);
        window.draw(solveButtonShape);
        window.draw(solveButtonText);
        // ...
        ```

    - **Panel principal de juego:**  
     Ocupa la mayor parte de la ventana y es donde se renderiza la cuadrícula triangular, el jugador y los elementos del laberinto. Aquí se dibuja el estado actual del juego y se gestionan las animaciones y movimientos.

        ```cpp
        // En main.cpp
        window.draw(gameGrid);
        window.draw(player);
        // ...
        ```

    Esta separación permite una experiencia visual clara y organizada, facilitando tanto la interacción como el seguimiento del progreso y las acciones dentro del juego.

## Sistema de Animaciones Basado en Tiempo

Para evitar transiciones bruscas y crear una experiencia de usuario fluida, todos los efectos visuales dinámicos se implementaron a través de un sistema de animación basado en tiempo, independiente de la velocidad de fotogramas del equipo.

**El Rol Central de `sf::Clock`:** La clase `sf::Clock` de SFML fue la herramienta fundamental, tal como se explicó su uso en la [sección anterior](graphics_lib.md). El patrón utilizado consistentemente fue:

1.  Iniciar un reloj (`restart()`) al comenzar una animación.
2.  En cada fotograma, calcular un valor de progreso (de 0.0 a 1.0) dividiendo el tiempo transcurrido (`getElapsedTime()`) por la duración total deseada.
3.  Utilizar este valor de progreso para interpolar linealmente la propiedad a animar (posición, opacidad, etc.).

**Animaciones Implementadas:**

- **Movimiento Suave del Jugador:** Para evitar la "teletransportación" entre celdas, se implementó una interpolación lineal. Al iniciar un movimiento, se almacenan las posiciones de inicio y destino. En cada fotograma, la posición visual del jugador se actualiza a un punto intermedio calculado con la fórmula: `posicion_actual = pos_inicio + (pos_destino - pos_inicio) * progreso`.

    ```cpp
    // En main.cpp
        if (isPlayerMoving) {
            float progress = playerMoveClock.getElapsedTime().asSeconds() / MOVE_DURATION_SECONDS;
            if (progress >= 1.0f) {
                isPlayerMoving = false;
                jugador.setPosicion(playerTargetPos);
            } else {
                jugador.setPosicion(playerStartPos + (playerTargetPos - playerStartPos) * progress);
            }
        }
    // ...
    ```

- **Visualización del Solver:** Cumpliendo con el requisito de una visualización "paso a paso" (*proporcionar visualización en tiempo real y paso a paso de los algoritmos*), la solución se presenta en dos fases animadas:

      1.  **Fase de Revelado:** Una primera animación (`animandoSolucion`) utiliza un `sf::Clock` para dibujar progresivamente los marcadores de la ruta, uno por uno, creando un efecto de "dibujado" del camino.
      2.  **Fase de Autoplay:** Una vez revelada la ruta, se inicia una segunda animación (`isAutoplaying`) donde el personaje del jugador sigue el camino revelado, utilizando la misma lógica de movimiento suave para cada paso.

        ```cpp
        // En GridModel.cpp
        void GridModel::dibujarSolucion(sf::RenderWindow &ventana, const std::vector<int> &camino, size_t numPasosAMostrar)
        {
        // ...
            sf::CircleShape marcador(5.f);
            marcador.setFillColor(sf::Color(231, 76, 60, 180));
            marcador.setOrigin({5.f, 5.f});
            for (size_t i = 0; i < numPasosAMostrar && i < camino.size(); ++i)
            {
                marcador.setPosition(getNodeCenter(camino[i]));
                ventana.draw(marcador);
            }
        }
        // ...
        ```

- **Transiciones de la Interfaz:** Para las pantallas de "GANASTE" y "PERDISTE", se utilizó un `sf::Clock` para animar el canal alfa (opacidad) del color del texto, logrando un efecto de desvanecimiento suave.
  ```cpp
  // En main.cpp
  // ...
      if (currentStatus != GameStatus::Jugando) {
          window.draw(overlay);
          const float FADE_DURATION_SECONDS = 1.5f;
          float progress = std::min(1.f, gameOverFadeClock.getElapsedTime().asSeconds() / FADE_DURATION_SECONDS);
          std::uint8_t alpha = static_cast<std::uint8_t>(255 * progress);
          sf::Color textColor;
  // ... (lógica para elegir color) ...
          textColor.a = alpha;
          gameOverText.setFillColor(textColor);
  // ...
  ```

## El Bucle de Renderizado

El orden de dibujado es crucial y se gestiona en el bucle principal de [`main.cpp`](src/main.cpp) para asegurar que los elementos se superpongan correctamente:

1.  `window.clear()`: Se limpia la pantalla con un color de fondo sólido.
2.  `miGrid.dibujar()`: Se dibuja la malla de triángulos.
3.  `miGrid.dibujarSolucion()`: Se dibuja la animación del camino solución (si está activa).
4.  `jugador.dibujar()`: Se dibuja al jugador sobre el grid.
5.  `window.draw(HUD)`: Se dibuja el **panel lateral de interfaz**, que incluye toda la información relevante para el jugador (puntuación, turnos, objetivo actual) y los botones de interacción como "Reiniciar" y "Resolver".
6.  `window.draw(Overlay)`: Se dibuja la capa de fin de juego (si está activa).
7.  `window.display()`: Se intercambia el búfer de renderizado con el visible, mostrando el fotograma final al usuario.

>Actualmente, como se mencionó anteriormente, la ventana se divide en dos paneles principales: el panel lateral de interfaz (HUD) y el panel principal donde se muestra el área de juego.

!!!note
    Para más información sobre como funcionan las clases de SFML, se puede consultar la sección [Biblioteca Gráfica y Manual de Uso](graphics_lib.md).
