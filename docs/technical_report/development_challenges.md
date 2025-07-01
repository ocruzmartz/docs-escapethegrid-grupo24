# **Desafíos de Desarrollo y Soluciones Implementadas**

Durante el desarrollo de "Escape the Grid", surgieron varios desafíos técnicos que requirieron soluciones específicas y un diseño cuidadoso. A continuación, se detallan los problemas más significativos y cómo fueron resueltos a nivel de código y arquitectura.

## Configuración de Dependencias Externas (SFML y CMake)

- **El Desafío:** Muchos usuarios pueden tener dificultades para instalar y configurar correctamente las bibliotecas externas requeridas, especialmente SFML y CMake, en diferentes sistemas operativos y entornos de desarrollo. Las rutas a los archivos de configuración de SFML (`SFMLConfig.cmake`) pueden variar, lo que provoca errores al compilar el proyecto si no se especifican correctamente.

- **La Solución:** Se documentó detalladamente el proceso de instalación y configuración de SFML y CMake en la guía de compilación. Además, se añadió una sección específica sobre cómo definir la variable `SFML_DIR` en caso de que CMake no detecte automáticamente la biblioteca. Esto permite a los usuarios adaptar la configuración a su entorno particular y evitar errores comunes de compilación.

## Problemas con el Entendimiento y Uso de SFML

- **El Desafío:** Durante las primeras etapas del desarrollo, uno de los principales retos fue comprender a fondo el funcionamiento de la biblioteca SFML, especialmente en lo referente al manejo de ventanas, eventos y renderizado eficiente de una malla de triángulos. La documentación oficial de SFML es extensa, pero algunos conceptos clave como la gestión de texturas, el ciclo de dibujo y la optimización de recursos gráficos no resultaron evidentes al principio. Esto llevó a errores como renderizados incompletos, parpadeos en pantalla y dificultades para detectar correctamente los clics del mouse sobre celdas triangulares.

- **La Solución:** Se invirtió tiempo en el estudio de ejemplos prácticos y en la experimentación directa con pequeños programas de prueba para aislar y entender cada componente de SFML. Se consultaron foros y tutoriales externos para aclarar dudas específicas sobre la detección de colisiones y la manipulación de vértices. Finalmente, se refactorizó el código de renderizado para aprovechar mejor las capacidades de SFML, logrando una visualización fluida y una interacción precisa con la malla triangular.

## Errores Humanos y Validaciones Insuficientes

- **El Desafío:** Durante el desarrollo, surgieron errores causados por validaciones insuficientes o mal implementadas en el código. Por ejemplo, en algunos casos no se comprobaba correctamente si los índices utilizados para acceder a vectores estaban dentro de los límites, lo que podía provocar errores de segmentación o cierres inesperados del programa. También hubo situaciones en las que no se validaba adecuadamente el estado del juego antes de permitir ciertas acciones del usuario, permitiendo movimientos no válidos o estados inconsistentes.

- **La Solución:** Se reforzaron las validaciones en puntos críticos del código, como el acceso a estructuras de datos y la gestión de los estados del juego. Se añadieron comprobaciones adicionales para evitar accesos fuera de rango y se implementaron mensajes de error claros para el usuario cuando se intentaba realizar una acción no permitida. Además, se revisó la lógica de avance y retroceso en la secuencia de la Caja Mágica para asegurar que el juego siempre se mantuviera en un estado coherente y seguro.


## A* con Restricciones Dinámicas (Paredes Temporizadas)

- **El Desafío:** El algoritmo A\* estándar está diseñado para operar sobre un grafo estático, donde los costos y la transitabilidad de los nodos no cambian. La mecánica de los `TIMED_WALL` (`T`) rompía esta premisa: una celda podía ser transitable al inicio de la búsqueda, pero convertirse en un muro intransitable en un futuro, dependiendo del número de turnos del camino encontrado. Un A* simple no podría prever este cambio, llevando a la generación de rutas inválidas.

- **La Solución:** La solución fue hacer que el algoritmo A* fuera "consciente del futuro". En lugar de solo verificar el estado actual de un nodo vecino, la implementación en [`Solver.cpp`](src/Solver.cpp) calcula el costo tentativo en turnos para llegar a ese vecino y lo suma a los turnos ya transcurridos. Este `turnoFuturo` es el que se utiliza para validar si una pared temporizada estaría activa en el momento de llegar a ella.

    ```cpp
    // ...existing code...
        for (int neighborId : gridNode.vecinos)
        {
            const auto &neighborNode = m_grid.getNodo(neighborId);
            float tentativeGCost = gCosts[currentNode.id] + 1;
            int turnoFuturo = turnosActuales + static_cast<int>(tentativeGCost);

            if (neighborNode.tipo == TipoCelda::PARED || 
                neighborNode.tipo == TipoCelda::VACIO || 
               (neighborNode.tipo == TipoCelda::TIMED_WALL && turnoFuturo >= TURN_LIMIT))
            {
                continue;
            }
    // ...existing code...
    ```
    Esta comprobación asegura que A* solo considere caminos que sean válidos durante toda su trayectoria, respetando el límite de `TURN_LIMIT` definido en [`config.hpp`](src/config.hpp).

## Resolución de Múltiples Objetivos Secuenciales (La "Caja Mágica")

- **El Desafío:** El algoritmo A\* está diseñado para encontrar la ruta óptima entre un único punto de inicio y un único punto de fin. El modo de juego "Caja Mágica" requería una solución para un problema de múltiples destinos secuenciales (Inicio -> Celda 1 -> Celda 2 -> Celda 3 -> Final). Una sola ejecución de A* era insuficiente.

- **La Solución:** Se implementó una capa de orquestación estratégica en la función `Solver::solve`. Esta función actúa como un planificador de alto nivel que utiliza A* como una subrutina.
    1.  Consulta el [`GameState`](src/GameState.hpp) para determinar la secuencia de objetivos restantes.
    2.  Ejecuta un bucle que llama a `findPath` para cada tramo del viaje (ej. desde la posición actual hasta la celda '1').
    3.  Acumula los turnos gastados en cada tramo y los pasa como parámetro (`turnosTranscurridos`) a la siguiente llamada de `findPath`.
    4.  Concatena los sub-caminos resultantes para formar una única ruta completa.

    ```cpp
    for (char finChar : waypoints_restantes)
    {
        // ... encontrar el nodo objetivo ...
        std::vector<int> subCamino = findPath(nodoDePartidaActual, *nodoFinOpt, turnosTranscurridos);

        if (subCamino.empty())
        {
            // ... manejar fallo ...
            return {};
        }

        turnosTranscurridos += subCamino.size() - 1;
        caminoCompleto.insert(caminoCompleto.end(), subCamino.begin() + 1, subCamino.end());
        nodoDePartidaActual = *nodoFinOpt;
    }
    // ...existing code...
    ```
    Este enfoque modular permitió resolver un problema complejo descomponiéndolo en una serie de búsquedas A* más simples, asegurando que la solución global fuera coherente con las reglas del juego.

## La Lógica de "Superayuda" y el Manejo de Estados Irresolubles

- **El Desafío:** ¿Qué debería ocurrir si el jugador se encierra a sí mismo y luego presiona el botón "Resolver"? Una llamada simple al `solver` desde la posición actual del jugador fallaría, resultando en una mala experiencia de usuario al no recibir ninguna ayuda.

- **La Solución:** Se implementó una lógica de "superayuda" con un mecanismo de repliegue (fallback) en [`main.cpp`](src/main.cpp).
    1.  **Intento Contextual:** Al presionar "Resolver", el juego primero intenta encontrar una solución desde la posición y turno actuales del jugador.
    2.  **Repliegue a Solución Óptima:** Si el intento anterior falla (es decir, `rutaSolucion` está vacía), el sistema no se rinde. En su lugar, crea una instancia temporal y limpia de `GridModel` y `GameState` para simular un reinicio. Llama al `solver` sobre este estado limpio para calcular la solución óptima global desde el inicio del juego.
    3.  **Presentación:** Finalmente, el juego principal se resetea y se utiliza la ruta óptima encontrada para iniciar el "autoplay", mostrando al jugador la mejor solución posible desde el principio.

    ```cpp
    // ...existing code...
                        rutaSolucion = solver->solve(jugador.getNodoActualId());

                        if (rutaSolucion.empty()) {
                            infoText.setString("Reiniciando... Mostrando solución óptima desde el inicio");
                            // --- LÓGICA CORREGIDA: Usamos un grid temporal para el cálculo ---
                            GridModel gridOptimo;
                            gridOptimo.cargarDesdeArchivo("mapa.txt", window);
                            GameState estadoInicial;
                            Solver solverDesdeInicio(gridOptimo, estadoInicial);
                            
                            std::vector<int> rutaOptima;
                            if(const auto startNodeId = gridOptimo.findStartNodeId()) {
                                rutaOptima = solverDesdeInicio.solve(*startNodeId);
                            }
                            if (!rutaOptima.empty()) {
                                setupGame(); // Resetea el juego a un estado limpio
                                rutaSolucion = rutaOptima; // Asignamos la ruta óptima encontrada
                            }
                        }
    // ...existing code...
    ```
    Esta solución robusta mejora drásticamente la experiencia de usuario, garantizando que el botón "Resolver" siempre ofrezca una respuesta útil.