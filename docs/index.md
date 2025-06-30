# Escape the Grid: The Magic Box

## Descripción General del Juego

**¡Bienvenido a Escape the Grid!** Un desafiante puzzle 2D estratégico donde cada movimiento cuenta y cada decisión es crucial. Atrapado en un laberinto de triángulos interconectados, deberás trazar tu camino hacia la libertad planificando cuidadosamente cada paso.

Lo que hace único a Escape the Grid no es solo su distintiva cuadrícula triangular, sino sus mecánicas que cambian constantemente el tablero. La salida permanece invisible ante tus ojos hasta que logres activar la misteriosa "Caja Mágica" pasando sobre las celdas amarillas en la secuencia correcta, la cual deberás descubrir por ti mismo. Pero cuidado: mientras exploras, ciertas celdas se transforman en muros impenetrables conforme avanzas, bloqueando posibles rutas de escape.

Comenzarás con 1000 puntos, pero cada vez que retrocedas por un camino ya recorrido, tu puntuación disminuirá. ¿Tienes la astucia para descifrar el patrón y encontrar la ruta perfecta hacia la libertad? ¿O quedarás atrapado para siempre en este laberinto cambiante?

<img src="img/game_image.png" alt="game image" width="80%" style="display: block; margin: 30px auto;"/>

## Características Principales

### Mecánicas del Juego
- **La Caja Mágica:** Descubre en el orden correcto pasando sobre las celdas amarillas. Solo completando esta misteriosa secuencia podrás hacer visible la salida.
- **Salida Oculta:** La meta permanece invisible hasta completar la secuencia correcta.
- **Paredes Temporizadas:** Las celdas azules se transforman en muros innacesibles después de 30 turnos.

- **Triangulación Dinámica:** Navega por un mapa compuesto enteramente de celdas triangulares interconectadas.

### Sistema de Puntuación
- **Puntos Iniciales:** 1000 puntos al comenzar.
- **Penalización por Retroceso:** -50 puntos cada vez que revisitas una celda.
- **Condición de Derrota:** Quedarte sin puntos o sin movimientos posibles.

### Interfaz y Controles
- **Control por Ratón:** Interacción completa mediante clics (clic izquierdo).
- **Panel Lateral Derecho:** Muestra puntuación actual, turnos restantes y próximo objetivo. Además de dos botones para reiniciar el juego o resolver automáticamente.
- **Panel Lateral Izquierdo:** Muestra el laberinto de triángulos donde se desarrolla la acción.
- **Visualización de Estado:** Feedback claro sobre victoria, derrota y progreso.

### Herramientas Avanzadas
- **Algoritmo de Resolución:** Encuentra y muestra la ruta óptima hacia la meta (usando el algoritmo A*).
- **Reproducción Automática:** Observa la solución ejecutada paso a paso.
- **Detección de Estado Final:** El juego reconoce automáticamente cuando te has quedado sin opciones.

## Información Adicional

En la sección Guía del Usuario encontrarás todo lo necesario para comenzar a jugar, desde la instalación hasta los controles y mecánicas del juego. En la sección Reporte Técnico encontrarás detalles sobre los algoritmos implementados y la biblioteca gráfica utilizada. Aquí tienes algunos accesos rápidos:

- Para más detalles sobre el algoritmo de resolución y cómo se implementa, revisa la sección de [Algoritmos Implementados](./technical_report/algorithms.md).

- Para más detalles sobre los controles y como jugar, revisa la sección de [Controles y Jugabilidad](./user_guide/how_to_play.md).

- Para más detalles sobre como configurar el entorno y compilar el juego, revisa la sección de [Instrucciones de Compilación](./user_guide/compilation.md).

- Para más detalles sobre cómo crear mapas personalizados, consulta la sección de [Mapas Personalizados](#mapas-personalizados).

- Para ver todas las mecánicas del juego en acción, revisa la sección de [Guía Visual](./user_guide/visual_guide.md).






