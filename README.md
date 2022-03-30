# Sequential-Ordering-Problem

## 1.-Introduccion ##

El presente documento plantea una solucion de programacion para, el problema de ordenamiento secuencial, en el cual se sigue la metodologia de resolverlo con el algoritmo de recocido simulado.


## 2.-Problema ##

### 2.1.-Definicion del problema ###

El problema de ordenamiento secuencial consiste en encontrar un camino hamiltonanio de costo minimo, esto quiere decir que se debe encontrar el camino con los menores costos. El grafo con el que se debe representar debe ser un grafo dirigido con costos asociados a los ejes y relaciones de precedencia entre los nodos. Al hablar de relaciones de presedencia, nos referimos a una o mas reglas en el grafo que indican que nodos se deben visitar primero.Este problema puede verse como una variante del TSP, la diferencia principal recae en que el grafo debe ser asincrono debido a la presedencia. Pero tampoco podemos decir que se trata de un problema ATSP.
De manera formal el problema se define de la siguiente forma;

Sea ![](https://latex.codecogs.com/svg.image?G=(V,E)) un grafo dirigido completo, donde ![](https://latex.codecogs.com/svg.image?V={0,1,2,3...}) es el conjunto de nodos y ![](https://latex.codecogs.com/svg.image?E={(i,j)|&space;i,j&space;E&space;V,/=j}.) Cada eje ![](https://latex.codecogs.com/svg.image?(i,j)&space;\exists&space;E) tiene un costo asociado ![](https://latex.codecogs.com/svg.image?Cij>=0.) Ademas, se define el grafo de precedencias ![](https://latex.codecogs.com/svg.image?P=(V,R)) con el mismo conjunto de nodos V .

Un camino que recorre todos los nodos sin repetirlos, comenzando en el 0 y terminando en n, y satisface todas las condiciones de precedencia es una solucion factible del SOP. El objetivo del SOP es encontrar una solucion factible de mınimo costo, donde el costo esta dado por la sumatoria de los costos de los ejes que componen el camino.


### 2.2 Aplicaciones ###
Algunas aplicaciones de este problema cuando encontramos alguna situacion como mejoramientos en cadenas de produccion, planificacion y ruteo por ejemplo:

**Planificacion de produccion:** Minimizar tiempo de ejecucion de varios trabajos, que deben ser proceso en cierto orden por una maquina.

**Optimizacion:** En el uso de una grua portuaria eliminando cuellos de botella.

**Problemas de transporte** por ejemplo minimizar la distancia recorrida por un helicoptero que debe transportar personal tecnico entre diferentes plataformas en una compañia petrolera.

**Optimizaciones en fabricas** por ejemplo en manufactura del automotor, en el sistema de pintado de los autos, para minimizar costos de cambio de color de la pintura.

### 2.3 Ejemplo ### 
**GRAFO DE EJEMPLO**

![Grafo de ejemplo.](https://graphonline.ru/tmp/saved/hh/hhFcwtLeeqzqzogz.png)  

**SOLUCION DE GRAFO DE EJEMPLO**

![Solucion de Grafo de ejemplo.](https://graphonline.ru/tmp/saved/gY/gYHqUXAsCVVxnMec.png)

**Vector Solucion**

[0, 2, 3, 4, 5, 6, 1, 7]

**Matriz de costo**


| |0 |1 |2 |3 |4 |5 |6 |7 |
|--|--|--|--|--|--|--|--|--|
|**0** |0 |1 |7 |2 |6 |7 |1 |5 |
|**1** |8 |0 |2 |3 |8 |4 | 6 | 7 |
|**2** |9 |1000 |0 |4 |1 |6 |9 |1 |
|**3** |6 |3 |3 |0 |3 |5 |4 |3 |
|**4** |8 |2 |3 |5 |0 |3 |7 |5 |
|**5** |1 |5 |6 |3 |8 |0 |6 |1 |
|**6** |7 |1000 |3 |8 |3 |6 |0 |9 |
|**7** |4 |6 |7 |8 |3 |2 |1 |0 |


## 3.- Modelo
### Función objetivo
La función objetivo para el problema de ordenamiento secuencial es la siguiente: ![](https://latex.codecogs.com/svg.image?\sum&space;C_{ij}&space;&plus;&space;(n&space;*&space;penalizacion))

Donde la **penalización** será igual al costo mayor de toda la matriz de costos y **n** representa el número de reglas de precedencia que no se cumplen.

El siguiente fragmento de código indica el proceso para calcular el valor de la función objetivo.

```Python
def obtenerCosto(solucion,costos,reglas):
    #obtener el total de nodos, se pone -1 porque no es necesario recorrer el ultimo
    nodos = len(solucion)-1 
    #Se obtiene el costo maximo de la tabla (es el que se usara para la penalización)
    costoMax = max(max(fila) for fila in a)
    costoTotal = 0
    # su suman los costos C de la solución
    for i in range(nodos):
        costoTotal += costos[solucion[i]][solucion[i+1]] 
    n = presedencia(solucion,reglas)
    costoTotal += n * costoMax
    return costoTotal
```
Para revisar las precedencias se utilizo la siguiente función

```Python
def presedencia(solucion,rules):
    #tiene que variar estos arreglos dependiendo de cuantas reglas aya.
    auxB = np.zeros(len(rules))
    cont = 0
    for i in range(len(solucion)):
        for j in range(len(rules)):       
            if rules[j][0] == solucion[i] :
                auxB[j]+=1       
            if auxB[j] == 1 :
                if rules[j][1] == solucion[i] :
                    auxB[j]+=1               
    for i in range(len(auxB)):
        if auxB[i] < 2 :
            cont+=1
    return cont
```

### Restricciones

Las restricciones con las que cuenta este problema se dividen en dos:
1. Las reglas de precedencia son aquella que indican si un nodo debe visitarse antes que otro, la forma en la que se visitan no es necesariamente consecutiva. Se representa de la siguiente manera: ![](https://latex.codecogs.com/svg.image?i&space;<&space;j&space;)
2. La solución que se genera debe de iniciar visitando el nodo cero y terminar con el nodo N

### Representación de la solución
Para poder generar la solución inicial, se debe realizar los siguientes pasos:
- Generar un arreglo de ![](https://latex.codecogs.com/svg.image?N&space;-&space;2) números aleatorios en un intervalo de ![](https://latex.codecogs.com/svg.image?[0,1])
- Ordenar de forma ascendente el arreglo de acuerdo al valor.
- Tomar los índices del arreglo ordenado para generar la solución inicial.
- Por último, para completar los nodos a visitar y cumplir con la segunda restricción se añade al inicio del arreglo el nodo ![](https://latex.codecogs.com/svg.image?0) y al final el nodo ![](https://latex.codecogs.com/svg.image?N)

Para el ejemplo que mostramos se tienen ![](https://latex.codecogs.com/svg.image?N=8), por lo que se generaron 6 números aleatorios, el vector sin ordenar es el siguiente: 

![](https://latex.codecogs.com/svg.image?0.4661691533999429&space;\to&space;1)

![](https://latex.codecogs.com/svg.image?0.9900212085683265&space;\to&space;2)

![](https://latex.codecogs.com/svg.image?0.4876416060768133&space;\to&space;3)

![](https://latex.codecogs.com/svg.image?0.6887360248200161&space;\to&space;4)

![](https://latex.codecogs.com/svg.image?0.027874235454644403&space;\to&space;5)

![](https://latex.codecogs.com/svg.image?0.7609657561616946&space;\to&space;6)

Una vez ordenado de forma ascendente  tenemos: 

![](https://latex.codecogs.com/svg.image?0.027874235454644403&space;\to&space;5)

![](https://latex.codecogs.com/svg.image?0.4661691533999429&space;\to&space;1)

![](https://latex.codecogs.com/svg.image?0.4876416060768133&space;\to&space;3)

![](https://latex.codecogs.com/svg.image?0.6887360248200161&space;\to&space;4)

![](https://latex.codecogs.com/svg.image?0.7609657561616946&space;\to&space;6)

![](https://latex.codecogs.com/svg.image?0.9900212085683265&space;\to&space;2)

Por lo que la representación de la solución inicial, una vez agregados los nodos inicial y final quedaría de la siguiente manera: ![](https://latex.codecogs.com/svg.image?[0,&space;5,&space;1,&space;3,&space;4,&space;6,&space;2,&space;7]) . Esto indica el orden en el que se deben ir visitando los nodos.

### Solución vecino
Una vez ya obtenida la solución inicial, para generar su solución vecino se debe realizar lo siguiente:
- Obtener dos números aleatorios entre 1 y N-2 (para no alterar el primer y último nodo), que representaran los indices de los nodos a intercambiar.
- Proceder a realizar el intercambio entre los nodos.

Números generados ![](https://latex.codecogs.com/svg.image?2) y  ![](https://latex.codecogs.com/svg.image?6)
Solución inicial: ![](https://latex.codecogs.com/svg.image?[0,&space;5,&space;1,&space;3,&space;4,&space;6,&space;2,&space;7]) Solución vecina: ![](https://latex.codecogs.com/svg.image?[0,&space;5,&space;1,&space;2,&space;4,&space;6,&space;3,&space;7])

## 4.- Instancias
El formato del archivo es csv en el que se encuentran almacenados la matriz de costos y la matriz de precedencia.

Dentro de este repositorio se encuentran 3 archivos, el primer archivo es un ejemplo con 50 nodos el segundo con 100 nodos y el tercero con 500 nodos.

