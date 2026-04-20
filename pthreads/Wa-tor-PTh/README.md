# Ejercicio: Wa-tor PTh.

## A rellenar por el alumno/grupo
 * Nombre y apellidos alumno 1   : <Antonio Jesús Lorenzo Ferrer >
 * Nombre y apellidos alumno 2   : <Adrián Pérez Pérez >
 * Nombre y apellidos alumno 3   : <David Martínez Hernández>
 * Nombre y apellidos alumno 4   : <Ivan Rojo Moreno>
 * Mayoría en GTA1, GTA2 o GTA3  : <GTA2>
 * Nombre de grupo de actividades: <GT2_07>


## Arquitectura: 
  * Microprocesador:Intel i5 10500
  * Número de núcleos: 6
  * Cantidad de subprocesos por nucleo: 2
  * Tiene hyperthreading (SMT) activado en BIOS: Está activado
  * HDD/SDD: 96GB
  * RAM: 15Gb
  * Se usa máquina virtual:NO
    - Número de cores:
    - RAM: 
    - Capacidad HDD: 


# Instrucciones:
 * Se pide realizar los **//TODO** del código. 
```console 
$ grep -n TODO *
```
  * En el Run.sh se tienen los comandos a ejecutar, (-r 201 -c 201)  y (-r 300 -c 300) con -ni 10000 y distinto número de hebras. 
**El número de iteraciones al final de la ejecución debe ser 10000**, es decir, deben quedar peces y tiburones después de la ejecución para obtener tiempos.
* No se usaran las salidas gráficas o de datos de gnuplot para medir los tiempos. Solo se usarán para chequear que el programa paralelo funciona como el secuencial.
* Para evitar algunas secciones criticas, se ha forzado que el tamaño del océano sea múltiplo de 3 en filas y columnas y se ha optado por la opción de tener tres bucles con i=0,1,2; i < rows; i=i+3 (y su respectivo bucle j para las columnas completo). **Cada hebra realiza una fila a la vez**.
* Las hebras tienen un barrier antes de empezar cada doble bucle (i,j) para que antes se ponga la *RowToProcess* a 0,1,o 2. Para 0 lo hace el main, y para 1 y 2 lo hace la hebra 0.
* Se ejecutan el main y -nt hebras.
* Nótese que las variables locales de la función de cada hebra son independientes en cada hebra.
* Se usa srand(0) y lrand48() para inicializar el océano por una sola hebra. 
* Se usa srand48\_r(i,&pRandData[i]) y lrand48_r(&pRandData[i],&Res) para el movimiento de peces y tiburones, así como la comida de tiburones. 
* Se entrega la versión secuencial (Wa-tor.c) y la paralela a modificar (Wa-tor-PTh.c).

## Librerias
   * El paquete **netpbm-progs** (o **netpbm**) para el comando del sistema rawtoppm.
   * El paquete **eog** para visualizar los ficheros .ppm.
   * El paquete **gnuplot** para generar las gráficas de evolución de peces y tiburones.
   * El paquete **valgrind** para chequear posibles memoria asignada y no liberada.
   * El paquete **ffmpeg** para generar video raw que se visualiza con ffplay


## Objetivos
 * Familiarizar al alumno con 
	- El uso de PThreads.
	- Trabajar con problemas dinámicos donde hay dependencia de datos.
	- Evitar secciones críticas con el desenrollado de bucles.
	- Usar mutexes y barriers.
	- Uso de secuencias de números aleatorios independientes por hebra.

## Compilación

```console 
$ make -j
$ make -j all
$ make -j Wa-tor
$ make -j wa-tor-PTh
```

## Ayuda parámetros 
```console
$ ./Wa-tor -h
$ ./Wa-tor-PTh -h
```
## Ejemplo de compilacion y ejecución
	En el script Run.sh

Wa-tor realiza una llamada a system para ejecutar **eog** (image viewer) si se le indica salida gráfica **-o** de los movimientos de los animales en el océano.

Wa-tor tambien usa el paquete **gnuplot** que hay que instalar si se le indica **-d** para general un fichero con número de peces y tiburones por iteración y mostrarlos como líneas en gratifico.

Wa-tor tambien usa el paquete **ffmpeg** que hay que instalar si se le indica el parámetro -ffmpeg para que la salida por el pipe generado la coja el comando ffmplay, generando un vídeo sin interrupciones del movimiento en el océano.

Antes de ejecutar Wa-tor hay que ver que no hay otra instancia corriendo:
```console
$ps aux  | grep Wa-tor
$ps aux  | grep eog
$ps aux  | grep gnouplot
$ps aux  | grep ffmplay

```
de **Wa-tor** o en su caso **eog**, **gnuplot**, y/o **ffmplay**. En tal caso matad el proceso necesario con:
```command 
$ kill -9 <pid>
```

- - -

# Entrega :

1. **¿Tiene el algoritmo paralelo finalizado perdidas de memoria (memory leaks)?**
 * Mostrar la salida de 
```console
$ valgrind -s ./Wa-tor-PTh -ni 102 -nt 4 
```
Si hay perdida de memoria es que dos animales se han movido a la misma celda y no está bien el código. Muestra aquí el ERROR SUMMARY.
En nuestro caso NO han habido pérdidas de memoria, aunque no nos los pidan el ERROR SUMMARY obtenido es el siguiente: 
==6238== HEAP SUMMARY:

==6238==   in use at exit: 0 bytes in 0 blocks

==6238==  total heap usage: 41,673 allocs, 41,673 frees, 755,982 bytes allocated

==6238== 

==6238== All heap blocks were freed -- no leaks are possible

==6238== 

==6238== ERROR SUMMARY: 0 errors from 0 contexts (suppressed: 0 from 0)


2. **Rellena la siguiente tabla:**
 * T.Sec es el tiempo de ejecución del algoritmo secencial.
 * T(p) es el tiempo de ejecución del algoritmo paralelo con -nt p hebras.

| Ejecución | 201x201   | 300x300    |
|-----------|-----------|------------|
|T.Sec      | 6,989s    | 15,932s    |
|T(1)       | 7,946s    | 17,296s    |
|T(2)       | 4,931s    | 10,977s    |
|T(4)       | 2,952s    | 6,463s     |


2. **¿Tiene ventajas su paralelización?** 
Si, al paralelizar, a parte de mejorar los tiempos en general, como se ve en la tabla, la imagen se genera en un orden aleatorio, lo que hace que no de la sensación que se genere de izquierda a derecha.

3. **¿Se podrían usar menos número de mutexes o barriers? ¿Porqué?**
Mutex no se pueden usar menos, porque los dos que hay son indispensables para sincronizar variables.
Barriers si se podrían usar menos, porque en vez de usar dos barreras una con el denotativo "starts" y otro "ends", puedes
usar una sola que haga ambas funciones. Actualmente hay dos solo para distinguir que hace cada uno para obtener una mayor
legibilidad, pero esto es opcional.

4. **¿Se te ocurre un método más eficiente de paralelizar Wa-tor?** 
En lugar de controlar las filas en formato "peine" y sincronizar las iteraciones alterminar un peine de filas, podríamos hacer que cada hilo manejen filas más cercanas haciendo peines más pequeños. Antes hacíamos las filas 0, 3, 6, 9 .... para evitar colisiones. Y si hacemos filas más juntas como 0,2,4,6,8.... de forma que controlemos las colisiones en el momento de producirse y redirigir el pez hacia otro sitio. Tendríamos menos peines pero mayor sincronización o granularidad más fina por lo que tendríamos que comprobar a nivel temporal a ver si es eficiente. 
RESUMEN: Menos peines-más sincronización.

5. **¿Has hecho un *make clean* y borrado todas los ficheros innecesarios (imágenes, etc) para la entrega antes de comprimir?**
Sí.
- - -

# Actividad Wa-tor: reglas.

Tomadas de [wiki](https://en.wikipedia.org/wiki/Wa-Tor)

## Movimento

* En cada iteración un pez o tiburón puede moverse norte sur este u oeste a un pixel adyacente, si no está ocupado por un miembro de su misma especie.

### Peces

* Si los 4 pixeles adyacentes están ocupados (por peces o tiburones), no se mueve.
* Se selecciona el pixel a moverse aleatoriamente entre los libres.

### Tiburón

* Se elige entre los adyacentes ocupados por peces, uno aleatoriamente y se come el pez.
* Si no hay peces adyacentes, se mueve como el pez, evitando otros tiburones.

## Vida y reproducción:

### Pez

Si el pez ha sobrevivido un número de iteraciones se reproduce. Esto ocurre cuando además hay un pixel adyacente libre en el cual se crea un nuevo pez. Ambos, padre e hijo ponen sus contadores de iteraciones a 0.

### Tiburón

* En cada iteración se le decrementa en 1 la energía.
* Al alcanzar la energía 0 el tiburón muere.
* Al comerse un pescado, el tiburón gana una cantidad de energía.
* Si el tiburón alcanza un número de iteraciones, se reproduce como el pez. El hijo coge la energía de creación de tiburón. El padre mantiene la suya.


- - - 

### Como ver este .md 
En linux instalar grip:

```console 
$ pip install grip 
```

y ejecutar
```console
$ grip -b README.md
```

### Markdown cheat sheet

[Markdown cheat sheet](https://www.markdownguide.org/cheat-sheet/)


- - -

&copy; [Leocadio González Casado](https://sites.google.com/ual.es/leo). Dpto, Informática, UAL.
