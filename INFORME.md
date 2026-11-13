# Informe: agentes para el 2048 simplificado

Integrantes: [Ofelia Gigena, Maximiliano Chavez, Guillermo Rojo]

## 1. Comparación

| Agente       | Puntuación media |  Ficha máxima (distribución) | Tiempo |
|--------------|------------------|------------------------------|--------|
| Al azar      |             19.26|4: 34.0%   8: 55.1%  16: 10.9%|  0.0 s | 
| Minimax, profundidad 1 |   57.14|8: 10.7%  16: 84.4%  32:  4.9%|  0.3 s |
| Minimax, profundidad 2 |   57.98|8:  9.3%  16: 85.6%  32:  5.1%|  1.7 s |
| Minimax, profundidad 3 |   57.98|8:  9.3%  16: 85.6%  32:  5.1%| 11.0 s |
| Q-learning,30000 episodios|66.46|8:  4.3%  16: 87.6%  32:  8.1%|  3.0 s |

El tiempo de Minimax es de evaluacion de 1000 partidas.El tiempo de 
q-learning es la suma del entrenamiento más la evaluacion. Este último tuvo 
mejor puntuación media y alcanzó la ficha 32 en el 8.1% de las partidas.
Minimax mejoró con la profundidad pero al pasar de 2 a 3 el tiempo aumentó
más de 6 veces! 

## 2. Minimax frente al azar real
Minimax es pesimista, supone que el otro jugador va a jugar en la peor posición 
o en la más desfavorable para él. No obstante, el juego original las fichas que van
apareciendo, lo hacen de forma aleatorio y además los 2 con probabilidad del 90% y
el 4 con probabilidad del 10%, de modo que Minimax resuelve un problema distinto.
En este sentido, Q-learning tiene la ventaja de aprender de las partidas que juega.
No necesita conocer las probabilidades.
En determinadas situaciones Minimax puede elegir, peor de lo que el azar puede 
hacer. Veamos un ejemplo: supongamos el siguiente tablero inicial
tablero
| . 4 | se puede mover a izquierda o arriba
| 2 8 | si movemos a la izquierda el tablero queda

| 4 . | Si aparece el 2 (90% probabilidad) el tablero se bloquea y el valor
| 2 8 | futuro es cero. Si aparece el 4 (10% probabilidad) se podría combinar
        los dos 4 el valor futuro sería 0,90 x 0 + 0,10 x 18 = 1,8

Si partiendo el tablero original movemos hacia arriba obtenemos:
| 2 4 | Si aparece el 2 (90% probabilidad) el tablero se pueden combinar los 2
| . 8 | y la función heurística asigna un valor de 16. Si aparece el 4 (10% 
       probabilidad) el juego termina y el valor futuro es cero, El valor 
       esperado es: 0.90 x 16 + 0.10 x 0 = 14.4 

Minimax asigna un valor 0 a ambas alternativas, porque toma siempre el peor
resultado posible, sin importar la probabilidad. Como es un empate, el programa
elige la primera acción según el orden definido en el problema: mover a izquierda
Así elige una alternativa cuyo valor esperado es 1,8, bastante menor que la otra 
14.4.-
Por otra parte, Q-learning aprende de los juegos que va experimentando y puede 
incorporar en sus estimaciones el comportamiento probabilístico del entorno.
Esto explica porque en el tablero 2x2 Q-learning obtuvo una puntuación media de 
66.46 frente a los 57.98 de Minimax (con profundidad 2). 
Esto lo que explica es que Q-learning se adapta mejor al azar real.


## 3. Profundidad
Cuando ejecutamos: py evaluate_agents.py obtenemos los resultados 
a continuación:

| Profundidad | Nodos expandidos por jugada | Puntuación media |
|-------------|-----------------------------|------------------|
|      1      |              7.0            |        57.14     |
|      2      |             54.9            |        57.98     |
|      3      |            359.6            |        57.98     |

La cantidad de nodos se obtiene de las llamadas a max_value acumuladas dividido 
por el total de jugadas. De la tabla se puede ver que, a medida que aumenta la 
profundidad los nodos, aumentan pero en la misma proporción la puntuacion media.
Entre prof. 2 a 3 la puntuación media no cambia pero la cantidad de nodos aumenta
notablemente. Esto ocurre porque no se memorizan los estados ya alcanzados (en 
esta implementación). Una búsqueda más profunda no garantiza mejorar la puntuación


## 4. Parámetros de Q-learning
En q_learning_agent el método def__init__ establece los parámetros: alpha, gamma y epsilon
Realizamos las siguiestes pruebas:  (>train_q_learning.py)
1) sin cambiar lo recibido alpha=0.1, gamma=1.0, epsilon=0.1
2) Mantenmos gamma y epsilon y usamos alpha= 0.03
3) Mantenmos gamma y epsilon y usamos alpha= 0.3
4) Mantenmos gamma y alpha=0.1 y usamos epsilon = 0.03
5) Mantenmos gamma y alpha=0.1 y usamos epsilon = 0.3

Resultados:
| Episodios | Base a=0.1 e=0.1 | a=0.03 e=0.1 | a=0.3 e=0.1 | a=0.1 e=0.03 | a=0.1 e=0.3 |
|-----------|------------------|--------------|-------------|--------------|-------------|
|     100   |       29.05      |        28.84 |       29.59 |        19.48 |       22.40 |
|     300   |       30.72      |        30.00 |       47.65 |        19.48 |       35.00 |
|    1000   |       52.84      |        30.23 |       57,94 |        41.68 |       53.65 |
|    3000   |       58.49      |        42.03 |       60,48 |        47.90 |       60.11 |
|   10000   |       61.36      |        57.65 |       66.90 |        56.73 |       62.06 |
|   30000   |       66.71      |        62.98 |       67.81 |        62.82 |       66.26 |
 
alpha controla la velocidad del aprendizaje, con un valor bajo como 0.03 aprende más lento 
porque modifica poco las estimaciones de cada experiencia. Con alpha 0.3 aprende más rápido
se y obtuvo la mayor puntuación. 
epsilon controla la exploración de alternativas, con un valor bajo como 0.03 explora poco, 
con valores más altos explora más. Esto puede favorece encontrar nuevas estrategias pero
a la vez puede probar acciones que no son mejores a las ya conocidas. En este experimento
con 0.3 alcanzó 66.50 de puntaje por debajo de 66.71 (configuración inicial)
En conclusión hay que probar distintos valores de estos parámetros para mejorar la
calidad del aprendizaje

## 5. Un tablero más grande
Resultados en 3 × 3
py evaluate_agents.py modificado con estos valores:
DEPTHS=[1] = 1   ROW= 3 COLS=3 GANES= 200

| Agente | Punt. media   |               Distribución de ficha máxima            | tiempo| 
|        |               |                                                       |       |           
|--------|---------------|-------------------------------------------------------|-------|
| Al azar|      179,92   |      4: 0.5%, 8: 3.5%, 16: 31.0%, 32: 57.5%, 64: 7.5% | 0.1 s |
| Minimax, profundidad 1 |      768,16   | 16: 0.5%, 32: 12.5%, 64: 48.0%, 128: 35.5%, 256: 3.5% | 3.0 s |
| Q-learning, 30 000 episodios 
|        |      206,02   |              8: 4.0%, 16: 31.0%, 32: 54.0%, 64: 11.0% | 16.4s |

Minimax supera ampliamente a los otros agentes en esta prueba, la heurística es mejor para tableros 
que tienen muchas fichas, alcanza una puntuación media de 768.16 y la mayor ficha es 256 en 3.5% de las 
partidas. La evaluación heurística y la búsqueda de movimientos es mucho mejor que las elecciones
aleatorias, en este caso. Q-learning superó al agente aleatorio en un tiempo considerablemente mayor.

py train_q_learning.py modificado modificado con estos valores:
ROW= 3 COLS=3
======================================================================
Q-learning on 2048 (3x3): alpha=0.1, gamma=1.0, epsilon=0.1
======================================================================
  episodes   mean score  Q entries  time (s)
       100       183.37       2729       1.0
       300       179.71       6896       1.8
      1000       181.17      19493       2.9
      3000       185.00      45879       4.6
     10000       198.63     109964       8.8
     30000       223.01     231395      19.9

Acá se ve que la puntuación mejora con el entrenamiento pero el aprendizaje resulta lento.
Q-learning rinde menos con 3x3 que con 2x2. Esto es porque en 3x3 aumentan los estados que el 
agente puede econtrar y por lo tanto la cantidad de pares estado-acción que debe aprender.
La tabla Q tuvo 231395 entradas después de 30000 episodios, en esta cantidad muchas situaciones 
se visitan pocas veces o nunca, las estimaciones de Q quedan incompletas, y la información para 
elegir correctamente es insuficiente.

Q-learning tiene más dificultades para aprender una política que resulte efectiva, debido a la
gran cantidad de estados posibles. Por otra parte Minimax obtiene mejores resultados porque 
evalúa las alternativas disponibles sin tener que aprender antes todos los posibles estados.


