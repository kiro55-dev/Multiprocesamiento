# Ejercicio: Mandel PTh.

## A rellenar por el alumno/grupo
 * Nombre y apellidos alumno 1   : <Antonio Jesús Lorenzo Ferrer >
 * Nombre y apellidos alumno 2   : <Adrián Pérez Pérez >
 * Nombre y apellidos alumno 3   : <David Martínez Hernández>
 * Nombre y apellidos alumno 4   : <Ivan Rojo Moreno>
 * Mayoría en GTA1, GTA2 o GTA3  : <GTA2>
 * Nombre de grupo de actividades: <GT2_07>

## Arquitectura: 
 * Microprocesador: 12th Gen Intel(R) Core(TM) i7-1255U
 * Número de núcleos: 10
 * Cantidad de subprocesos por núcleo: 2
 * Tiene hyperthreading (SMT) activado en BIOS: Si
 * HDD/SDD: SSD
 * RAM: 16GB
 * Se usa máquina virtual: Si
    - Número de cores: 4
    - RAM: 4GB
    - Capacidad HDD: 25GB

## Instrucciones:

* El Mandel.c muestra como generar una imagen en color del fractal de Mandelbrot.
> Información del fractal de Mandelbrot en la [wiki](https://es.wikipedia.org/wiki/Conjunto_de_Mandelbrot). 
>
> Mandelbrot set [Rosetta code](https://rosettacode.org/wiki/Mandelbrot_set#C)
>
> Mandelbrot set  [Techniques for computer generated pictures](https://www.math.univ-toulouse.fr/~cheritat/wiki-draw/index.php/Mandelbrot_set) 
* El fractal de Mandelbrot es en blanco y negro aunque se pueden generar tonos de gris o colores dependiendo del número de iteraciones "NIter" que se realicen sobre cada punto (pixel).
* Se proporciona el código que genera el fractal de mandelbrot en color usando la rutina NIterToRGB(). 
* Los parámetros se introducen por la línea de comandos.
* La memoria se asigna de forma dinámica. Aunque se podría escribir directamente en el fichero de salida. Se usan:
    +  Tres matrices (R, G y B) para generar los colores RGB de cada pixel.
 * En el Run.sh se tienen los comandos a ejecutar para -mi 1e4 y -mi 1e5 para el algoritmo secuencial (Mandel) y el paralelo (Mandel-PTh) con distinto número de hebras.
    - La versión secuencial se usará para obtener T.Sec y T.CsPar.
    - La versión paralela con p hebras se usará para obtener T(p).
 * En la versión paralela se ha añadido las opciones:
    + -cs < chunck size > por defecto 1.
    + -nt < número de hebras > 
 * La opción **-o** genera la imagen. La imagen se debe usar para verificar que el código es correcto, al visualizarse la imagen con el comando *eog*.
 * Se requiere completar el Mandel-PTh. Ver **//TODO** en el código.
 * Las hebras en Mandel-PTh trabajan por filas completas. Cada hebra trabajará en un número ChunkSize de filas indicado en la opción -cs (chunk size), que por defecto es de una fila cada vez.
 * De Mandel OMP sabemos que una distribución estática, donde cada hebra realiza Rows/NThreads filas, no es lo más eficiente debido a la distinta carga computacional por pixel y por tanto también por filas. Por otro lado, con -cs 1, donde cada hebra realiza una fila cada vez, las hebras tienen más probabilidad de esperar en el mutex para saber en qué fila comienzan la nueva ejecución, que si se usaran valores mayores de ChunkSize.
 * Cuando se midan tiempos PRINT=0 y DEGUD=0.

## Librerías
Se necesita tener instalados los siguientes paquetes:
  * netpbm-progs (o netpbm) para los comandos del sistema rawtoppm y rawtopng.
  * eog para visualizar la imagen.

## Objetivos
 * Familiarizar al alumno con 
	- El uso de PThreads.
 * Usar balanceo de la carga variando tamaño de chunck para problemas con diferente carga computacional por iteración.
 * Familiarizarse con el uso de exclusión mutua al modificar variables compartidas entre hebras (siguiente fila a computar).
  * Cómo medir el tiempo consumido de CPU, Wall-clock-time y el speed-up.

## Compilación

```console 
$ make -j
$ make -j Mandel
$ make -j Mandel-PTh
$ make -j all
```

## Ayuda parámetros 
```console
$ ./Mandel -h
$ ./Mandel-PTh -h
```

## Ejemplo de compilacion y ejecución
	En el script Run.sh

- - -

# Entrega:

## Speed-up teórico

1. **Rellena la siguiente tabla para la versión secuencial, Mandel.c.**


| Ejecución   | -mi 1e4         | -mi 1e5           |
| ----------- | --------------- | ----------------- |
|T.Sec        |   4,571s        |      42,768s      |
|T.CsPar      |   4,4697s       |      42,6715s     |
|SpA(2)       |   1,95663       |      1,9955       |
|SpA(4)       |   3,75064       |      3,973        |

donde
 * T.Sec: El wall-clock time (tiempo total) del programa secuencial. Parte real del $time Mandel ... 
 * T.CsPar: El tiempo de la parte del código secuencial que será paralelizado: doble bucle.
 * SpA(p): El spedd-up **teórico** según la ley de Amhdal para p hebras paralelas.



## Speed-up real Sp(p): 
 * Para las siguientes tablas, todo se mide para el algoritmo paralelo Mandel-PTh.c. T.Sec es el obtenido en el punto 1.
 * T(p): Wall clock time del programa paralelo con p hebras.
 * Sp(p): ganancia en velocidad con p hebras.

2. **Con -cs 1. Es el valor por defecto.**

| Ejecución   | -mi 1e4         | -mi 1e5         |
| ----------- | --------------- | --------------- | 
|T.Sec        |   4,571s        |      42,768s    | 
|T(1)         |   4,488s        |      42,544s    |
|T(2)         |   2,709s        |      23,344s    | 
|T(4)         |   1,643s        |      15,147s    | 
|Sp(1)        |   1,018         |      1,00526    | 
|Sp(2)        |   1,687         |      1,8276     | 
|Sp(4)        |   2,782         |      2,8235     |

3. **Con -cs Rows/(p=NThreads). Distribución estática.**

| Ejecución   | -mi 1e4         | -mi 1e5         |
| ----------- | --------------- | --------------- | 
|T.Sec        |      4,571s     |     42,768s     | 
|Chuck p=2    |      512        |     512         |
|Chunk p=4    |      256        |     256         |
|T(2)         |      2,667s     |     23,810s     | 
|T(4)         |      2,512s     |     43,411s     | 
|Sp(2)        |      1,7139     |     1,7962      | 
|Sp(4)        |      1,8196     |     0,9851      | 


4. **Con el mejor valor encontrado para -cs.**
* Hay que hacer una búsqueda dicotómica del mejor chunk.
* Rows = número de filas totales a paralelizar.
* Se obtiene T(p).max con el chunk max = máximo chunk = Rows/p
* Se obtiene T(p).min con el chunk min = mínimo chunk = 1.
* Repetir
    - Se obtiene T(p).med con el chunk med = (max - min)/2
    - Si T(p).min < T(p).max
       + max=med
    - Si no 
       + min=med
* hasta que T(p).min sea similar a T(p).max o min=max+1.
* Resultado: Chunck= min o max que tenga menor valor de T(p).Chunk.

**Resumiendo**
Se continua en el intervalo [min-cs,med-cs] o [med-cs,max-cs] que tenga mejores tiempos 
T.(min-cs), T.(med-cs) o T.(max-cs). 

Búsqueda para p=2.
======================================
1ªBusqueda                    Tiempos
T(p).max = 1024/2 = 512     ->23.810s 
T(p).min = 1                ->23.344s
T(p).med = 126
======================================
======================================
2ªBusqueda                    Tiempos
T(p).max = 256              ->23.822s 
T(p).min = 1                ->23.344s
T(p).med = 128
======================================
======================================
3ªBusqueda                    Tiempos
T(p).max = 128              ->23,661s
T(p).min = 1                ->23.344s
T(p).med = 32
======================================
======================================
4ªBusqueda                    Tiempos
T(p).max = 64               ->23.507s
T(p).min = 1                ->23.344s
T(p).med = 16
======================================
======================================
5ªBusqueda                    Tiempos
T(p).max = 32               ->23.466s 
T(p).min = 1                ->23.344s
T(p).med = 16               
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 16               ->23.523s
T(p).min = 1                ->23.344s
T(p).med = 8
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 8                ->23.377s
T(p).min = 1                ->23.344s
T(p).med = 4
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 4                ->23.351s
T(p).min = 1                ->23.344s
T(p).med = 2
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 2                ->23.363s
T(p).min = 1                ->23.344s
T(p).med = 1
======================================
El mejor chunck es 1

Búsqueda para p=4
Se empieza desde el chunk más alto posible, que es 1024/4 = 256.
======================================
1ªBusqueda                    Tiempos
T(p).max = 1024/4 = 256     ->32.007s 
T(p).min = 1                ->15.147s
T(p).med = 126
======================================
======================================
2ªBusqueda                    Tiempos
T(p).max = 128              ->21.076s 
T(p).min = 1                ->15.147s
T(p).med = 64
======================================
======================================
3ªBusqueda                    Tiempos
T(p).max = 64               ->15.123s
T(p).min = 1                ->15.147s
T(p).med = 32
======================================
======================================
4ªBusqueda                    Tiempos
T(p).max = 32               ->15.375s
T(p).min = 1                ->15.147s
T(p).med = 16
======================================
======================================
5ªBusqueda                    Tiempos
T(p).max = 16               ->14.983s 
T(p).min = 1                ->15.147s
T(p).med = 8               
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 12               ->14.924s
T(p).min = 8                ->14.875s
T(p).med = 10
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 10               ->14.820s
T(p).min = 8                ->14.875s
T(p).med = 9
======================================
======================================
6ªBusqueda                    Tiempos
T(p).max = 9                ->14.842s
T(p).min = 9                ->14.842s
T(p).med = 9
======================================
El mejor chunk es de 10


De todas formas, si uan hebra hace una fila entera, el mejor valor del chunk ( -cs ) se encuentra en el intervalo [1,32]. Búscalo.

**¿Por qué calcular el mejor chunk?**
* Con cs=1 el número de filas a repartir entre subprocesos = 1024. Determinar la fila y el subproceso consume tiempo.
* Con cs=Rows/p el número de repartos es el más pequeño pero habrá desbalanceo de la carga computacional.
* Por lo tanto, hay que encontrar un equilibrio entre ambos.


| Ejecución   | -mi 1e5         |
| ----------- | --------------- | 
|T.Sec        |     42,768s     | 
|Chuck p=2    |     1           |
|Chunk p=4    |     10          |
|T(2)         |     23,344s     | 
|T(4)         |     14,820s     | 
|Sp(2)        |     1,8320      | 
|Sp(4)        |     2,8858      | 


5. **Compara el mejor Sp(4) para -mi 1e5 de Mandel-OMP con schedule dynamic y el mejor chunk encontrado con Mandel-PTh del punto 4.**
 + **¿Cual es mejor?**
 + **¿Son los chunks OMP y PTh distintos (indica sus valores)?**
En teoría debería ser mejor la ejecución con el mejor chunk de Mandel-Pth, partimos de la base de que PThreads es muchísimo más rápido y eficiente que OpenMP y en el nuestro caso usando el chunk 10 hemos obtenido un Sp(4)PTh= 2,8858, comparado con el que obtuvimos en la práctica anterior que fue de Sp(4)OMP= 4,054 ES PEOR, pero debido a que la practica de MandelOmp obtuvimos resultados un poco irreales de bido a que fue ejecutado en máquina virtual, y los resultados pueden variar mucho.
 + ¿Son los chunks OMP y PTh distintos (indica sus valores)?
   Valor de chunck en OMP -> 4 para p=2 y 3 para p=4
   Valor de chunck en PTh -> 1 para p=2 y 10 para p=4

6. **Indica al número de filas que realiza cada hebra para una ejecución con p=4 de la tabla en el punto 4.**
    - **¿Difieren los números de filas realizadas por cada hebra de una ejecución a otra? ¿Porqué?**
    - **¿Es el número de filas realizado por las hebras de una ejecución similar? ¿Porqué?**
* Respuesta: El número de filas que realiza cada hilo es diferente en cada ejecución, todos los hilos no van a realizar el mismo número de filas, ya que unas tendrán una carga superior a la de otras filas. Puede que en lo que un hilo está ejecutando una fila con más carga, a otro hilo le de tiempo a ejecutar muchas más filas con una carga más leve. Por esto, cada hilo ejecutará una cantidad de filas diferente dependiendo de la carga y variando en cada ejecución. El número será muy diferente entre cada hilo, ya que unos pueden realizar muchas filas y otros muy pocas según como se reparta la carga.


7. **En vez de solo por filas, se podría haber paralelizado por pixeles. Teóricamente, ¿sería más efectivo? ¿Cuando y porqué?**
* Para ello, el pixel donde empezar se numera de 0 a (Rows * Cols)-1. Para pasar de número de pixel a posición [i][j] se puede usar la rutina VectorToMatrixInd() que se encuentra en el fichero Indexes-Vector-Matrix.c, y este en Rutines-PTh.tgz, que está en el aula virtual de la asignatura. Solo habría que usar VectorToMatrixInd() para el primer pixel del chunk, ya que los demás se pueden saber teniendo en cuenta Rows, Cols y el ChunkSize.

* Respuesta: Al fin y al cabo, se deberían de repartir píxel a píxel para cada hilo en lugar de cada fila de la imagen. Esto supone pagar un coste temporal mayor en la gestión y repartición de carga entre cada hilo. Pero podemos ganar a nivel de tiempo total, ya que hay filas que apenas tienen píxeles y sería más conveniente usar la parelización por píxeles en este caso que tenemos trabajos tan desproporcionados para cada hilo.

8. **¿Has hecho un *make clean* y borrado todas los ficheros innecesarios (imágenes, etc) para la entrega antes de comprimir?**
Sí.

- - -
### Cómo ver este .md 
En linux instalar grip:

```console 
$ pip install grip 
```

y ejecutar
```console
$ grip -b README.md
```

### Markdown cheat sheet

Para añadir información a este README.md:

[Markdown cheat sheet](https://www.markdownguide.org/cheat-sheet/)

- - -

&copy; [Leocadio González Casado](https://sites.google.com/ual.es/leo). Dpto, Informática, UAL.
