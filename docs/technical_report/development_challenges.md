# **Desafíos de Desarrollo y Soluciones Implementadas**

Durante el desarrollo de "Escape the Grid", surgieron varios desafíos técnicos que requirieron soluciones específicas y un diseño cuidadoso. A continuación, se detallan los problemas más significativos y cómo fueron resueltos a nivel de código y arquitectura.

## **1. A* con Restricciones Dinámicas (Paredes Temporizadas)**

- **El Desafío:** El algoritmo A* estándar está diseñado para operar sobre un grafo estático, donde los costos y la transitabilidad de los nodos no cambian. La mecánica de los `TIMED_WALL` (`T`) rompía esta premisa: una celda podía ser transitable al inicio de la búsqueda, pero convertirse en un muro intransitable en un futuro, dependiendo del número de turnos del camino encontrado. Un A* simple no podría prever este cambio, llevando a la generación de rutas inválidas.

- **La Solución:** La solución fue hacer que el algoritmo A* fuera "consciente del futuro". En lugar de solo verificar el estado actual de un nodo vecino, la implementación en [`Solver.cpp`](src/Solver.cpp) calcula el costo tentativo en turnos para llegar a ese vecino y lo suma a los turnos ya transcurridos. Este `turnoFuturo` es el que se utiliza para validar si una pared temporizada estaría activa en el momento de llegar a ella.

    ```cpp
    // filepath: c:\Users\mosca\Escritorio\Ciclo 01-25\TSC\test_repo\tsc-project-game\src\Solver.cpp
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

## **2. Resolución de Múltiples Objetivos Secuenciales (La "Caja Mágica")**

- **El Desafío:** El algoritmo A* está diseñado para encontrar la ruta óptima entre un único punto de inicio y un único punto de fin. El modo de juego "Caja Mágica" requería una solución para un problema de múltiples destinos secuenciales (Inicio -> Ítem 1 -> Ítem 2 -> Ítem 3 -> Final). Una sola ejecución de A* era insuficiente.

- **La Solución:** Se implementó una capa de orquestación estratégica en la función `Solver::solve`. Esta función actúa como un planificador de alto nivel que utiliza A* como una subrutina.
    1.  Consulta el [`GameState`](src/GameState.hpp) para determinar la secuencia de objetivos restantes.
    2.  Ejecuta un bucle que llama a `findPath` para cada tramo del viaje (ej. desde la posición actual hasta el Ítem '1').
    3.  Acumula los turnos gastados en cada tramo y los pasa como parámetro (`turnosTranscurridos`) a la siguiente llamada de `findPath`.
    4.  Concatena los sub-caminos resultantes para formar una única ruta completa.

    ```cpp
    // filepath: c:\Users\mosca\Escritorio\Ciclo 01-25\TSC\test_repo\tsc-project-game\src\Solver.cpp
    // ...existing code...
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

## **3. La Lógica de "Superayuda" y el Manejo de Estados Irresolubles**

- **El Desafío:** ¿Qué debería ocurrir si el jugador se encierra a sí mismo y luego presiona el botón "Resolver"? Una llamada simple al `solver` desde la posición actual del jugador fallaría, resultando en una mala experiencia de usuario al no recibir ninguna ayuda.

- **La Solución:** Se implementó una lógica de "superayuda" con un mecanismo de repliegue (fallback) en [`main.cpp`](src/main.cpp).
    1.  **Intento Contextual:** Al presionar "Resolver", el juego primero intenta encontrar una solución desde la posición y turno actuales del jugador.
    2.  **Repliegue a Solución Óptima:** Si el intento anterior falla (es decir, `rutaSolucion` está vacía), el sistema no se rinde. En su lugar, crea una instancia temporal y limpia de `GridModel` y `GameState` para simular un reinicio. Llama al `solver` sobre este estado limpio para calcular la solución óptima global desde el inicio del juego.
    3.  **Presentación:** Finalmente, el juego principal se resetea y se utiliza la ruta óptima encontrada para iniciar el "autoplay", mostrando al jugador la mejor solución posible desde el principio.

    ```cpp
    // filepath: c:\Users\mosca\Escritorio\Ciclo 01-25\TSC\test_repo\tsc-project-game\src\main.cpp
    // ...existing code...
                        rutaSolucion = solver->solve(jugador.getNodoActualId());

                        if (rutaSolucion.empty()) {
                            infoText.setString("No hay solucion desde aqui. Mostrando solucion optima...");
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