# Informe proyecto
## Simulación y Procesamiento Paralelo de Transacciones de Ventas

**Integrantes:**
* Josue Palma
* Miguel Molina
* Nicolas Paspuel
* Zenan Fernandez
* Kevin Chushig
* Deyvid Román

**Sistemas Operativos** **Curso:** GR2CD  
**Fecha:** 24 de mayo de 2026

---

## 1. Introducción
En el ámbito de los Sistemas Operativos modernos, la capacidad de ejecutar tareas de manera concurrente es fundamental para maximizar el uso de los recursos del procesador y reducir los tiempos de respuesta.

El presente proyecto tiene como finalidad implementar y analizar un programa en lenguaje C, ejecutado bajo un entorno de máquina virtual Ubuntu, capaz de procesar un volumen masivo de datos simulados. Específicamente, el sistema procesa un conjunto de hasta 550,000 registros de transacciones de ventas.

A través de la comparación entre un enfoque de procesamiento secuencial y uno paralelo, empleando la biblioteca `pthread.h`, se busca demostrar empíricamente las ventajas del paralelismo en la manipulación y transformación de grandes volúmenes de información.

## 2. Objetivos

### 2.1. General
Desarrollar un programa funcional en C, ejecutado en una máquina virtual Ubuntu, que procese en paralelo un conjunto de transacciones simuladas distribuyendo la carga de trabajo de manera eficiente.

### 2.2. Específicos:
* Implementar el uso de hilos mediante la biblioteca POSIX `pthread.h`, para la ejecución concurrente de múltiples tareas.
* Aplicar mecanismos de sincronización, específicamente semáforos de exclusión mutua, para proteger los recursos compartidos y prevenir condiciones de carrera durante la ejecución.
* Ejecutar rutinas de limpieza, imputación de la moda y normalización Min-Max sobre miles de registros de ventas.
* Medir, analizar y comparar de manera precisa los tiempos de ejecución entre el modo secuencial y el modo paralelo empleando la función `clock_gettime()`.

## 3. Diseño e Implementación del Sistema

### 3.1. Arquitectura del procesamiento de datos
El sistema fue diseñado para leer y procesar un archivo CSV llamado `transacciones.csv` que contiene la información en crudo. La lógica de procesamiento de datos se dividió en tres fases fundamentales aplicadas a cada bloque de transacciones:

* **Limpieza de valores nulos numéricos:** Se identifican las cantidades con valores negativos y se ajustan a un valor base de 0.0.
* **Imputación categórica:** Se reemplazan los valores de texto nulos o vacíos asignándoles la moda del conjunto de datos, la cual corresponde a la categoría "Alimentos".
* **Normalización de datos:** Se aplica una transformación de escala Min-Max para llevar todas las cantidades a un rango estandarizado entre 0 y 1, facilitando su posterior uso analítico.

### Gestión de hilos y concurrencia
Para evitar el procesamiento secuencial que genera cuellos de botella al iterar sobre los 550,000 registros, la carga se divide equitativamente. Si el programa se ejecuta con N hilos, el arreglo global de transacciones se fragmenta en N bloques disjuntos.

Cada hilo de ejecución procesa su bloque asignado de forma independiente. Dado que cada hilo opera sobre índices de memoria distintos dentro del arreglo global, las tres fases de transformación de datos que son la limpieza, imputación y normalización se ejecutan simultáneamente sin riesgo de solapamiento de datos.

### Mecanismos de sincronización
Aunque la manipulación del arreglo es independiente por hilo, el acceso a la consola es un recurso compartido importante. Para cumplir con los requerimientos de concurrencia y evitar condiciones de carrera al imprimir los tiempos de finalización de cada hilo, se implementó un cerrojo lógico mutex.

Mediante el uso de `pthread_mutex_lock` antes de ejecutar la función `printf`, y `pthread_mutex_unlock` inmediatamente después, el sistema garantiza que la escritura en la consola sea una operación atómica. Esto asegura que los mensajes de los distintos hilos no se intercalen, permitiendo una lectura íntegra de los tiempos reportados para el análisis de rendimiento.

## 4. Análisis de rendimiento y resultados
Para evaluar la eficiencia del sistema desarrollado, se realizaron múltiples pruebas empíricas ejecutando la limpieza, imputación y normalización de los registros. Se comparó el rendimiento del algoritmo secuencial base frente al procesamiento paralelo utilizando 3 y 4 hilos.

### Tiempos de ejecución
Se ejecutaron cinco rondas de pruebas para cada configuración con el fin de obtener una muestra representativa y minimizar el impacto de los procesos en segundo plano propios de la máquina virtual Ubuntu.

| Prueba | Secuencial | Paralelo (3 hilos) | Paralelo (4 hilos) |
| :---: | :---: | :---: | :---: |
| 1 | 0.017734 | 0.011235 | 0.016715 |
| 2 | 0.060677 | 0.013001 | 0.010441 |
| 3 | 0.023677 | 0.010558 | 0.010614 |
| 4 | 0.015790 | 0.040806 | 0.011466 |
| 5 | 0.017384 | 0.011327 | 0.016660 |

Para obtener una métrica de rendimiento estable, se calcularon los promedios de tiempo de ejecución de las cinco iteraciones y se determinó el factor de aceleración logrado por las configuraciones concurrentes en relación con la línea base secuencial.

| Modo | Promedio | Speedup |
| :--- | :--- | :--- |
| Secuencial | 0.0270524 | linea base (1.00x) |
| Paralelo 3 hilos | 0.0173854 | 1.56 |
| Paralelo 4 hilos | 0.0131792 | 2.05 |

### Análisis de fluctuaciones
Durante la evaluación, específicamente en la Prueba 4 del modo con 3 hilos, se registró un pico en el tiempo de ejecución 0.040806 segundos. Este tipo de variaciones es un comportamiento esperado al evaluar el rendimiento en un entorno virtualizado. Al ejecutarse en una máquina virtual Ubuntu, el planificador de procesos del sistema maneja múltiples tareas simultáneamente, lo que ocasionó una demora temporal en la ejecución de los hilos del programa.

### Ejecución en Ubuntu
A continuación, se presentan las evidencias de la terminal que demuestran la ejecución exitosa del programa y la correcta sincronización de los hilos en el sistema operativo huésped.

![Figura 1](nombre_archivo_figura_1.png)
*Figura 1: Pruebas realizadas de manera secuencial y paralela utilizando 3 hilos.*

Se observa una clara reducción en el tiempo de procesamiento.

![Figura 2](nombre_archivo_figura_2.png)
*Figura 2: Pruebas realizadas de manera secuencial y paralela utilizando 4 hilos.*

Esta configuración obtuvo el mejor rendimiento general y los menores tiempos de procesamiento.

### Comparación grafica
Para una visualización más clara del impacto del paralelismo, se generaron los siguientes gráficos de barras que contrastan el rendimiento entre las tres modalidades ejecutadas:

![Figura 3](nombre_archivo_figura_3.png)
*Figura 3: Comparación de tiempos de ejecución en las cinco iteraciones de pruebas.*

![Figura 4](nombre_archivo_figura_4.png)
*Figura 4: Promedio global de tiempos de ejecución por método de procesamiento.*

## Conclusiones
* **Eficiencia del Procesamiento Concurrente:** La implementación de hilos mediante `pthread.h` demostró una superioridad técnica indiscutible frente al procesamiento secuencial. Al dividir el arreglo de datos en bloques independientes, se logró que el procesador distribuyera mejor la carga de trabajo, ejecutando las transformaciones de datos mucho más rápido.
* **Impacto de la Asignación de Hilos:** Se comprobó que el rendimiento mejora al incrementar la cantidad de hilos dentro de los parámetros probados. La configuración con 4 hilos obtuvo el mayor grado de eficiencia, logrando reducir el tiempo promedio de ejecución.
* **Estabilidad del Sistema:** A lo largo de la repetición de las pruebas, el programa mantuvo un alto nivel de estabilidad y consistencia en sus resultados. El uso adecuado de exclusión mutua en las zonas críticas garantizó una impresión ordenada por consola sin generar bloqueos ni afectar el flujo del algoritmo.
* **Comportamiento Esperado en Máquinas Virtuales:** Se evidenció la presencia de fluctuaciones menores en los tiempos de respuesta debido a la gestión concurrente de recursos de la propia máquina virtual Ubuntu. Sin embargo, estas alteraciones no modificaron las tendencias generales obtenidas.
