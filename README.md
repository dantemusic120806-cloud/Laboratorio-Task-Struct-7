# Laboratorio-Task-Struct-7
# Informe de Laboratorio: Sincronización y Concurrencia
## El Problema de la Sección Crítica: De la Teoría del Software a la Realidad del Hardware

## 1. Introducción y la Condición de Carrera
Cuando múltiples hilos de ejecución comparten un espacio de memoria y escriben de forma concurrente sobre una misma variable global (por ejemplo, un contador incremental `contador++`), el resultado obtenido suele diferir drásticamente del valor esperado.

A nivel de lenguaje ensamblador, una operación aparentemente simple como `contador++` no es una instrucción atómica, sino que se divide en tres pasos secuenciales:
1. **LOAD:** Cargar el valor actual de la variable desde la memoria RAM hacia un registro interno de la CPU.
2. **MODIFY:** Incrementar el valor dentro del registro.
3. **STORE:** Almacenar el resultado de vuelta en la posición de memoria compartida.

Si el planificador del sistema operativo interrumpe a los hilos entre estos pasos, se produce un entrelazado impredecible conocido como **Condición de Carrera (*Race Condition*)**, donde las escrituras de un hilo sobreescriben y destruyen los cómputos de los demás, generando resultados no deterministas.

---

## 2. El Marco Teórico de la Sección Crítica y las Tres Leyes
Para garantizar la integridad de los datos compartidos, el código que accede a dichos recursos debe aislarse en una estructura lógica denominada **Sección Crítica**, la cual se compone de cuatro fases obligatorias:
1. **Sección de Entrada:** Mecanismo donde un hilo solicita permiso exclusivo para ingresar.
2. **Sección Crítica:** Bloque de código protegido donde solo un hilo puede operar a la vez.
3. **Sección de Salida:** Código encargado de liberar el permiso o candado.
4. **Sección Restante:** El resto del código que no interactúa con recursos compartidos.

Toda solución robusta de sincronización debe cumplir estrictamente con **tres leyes fundamentales**:
* **Exclusión Mutua:** Si un hilo se encuentra ejecutando dentro de su sección crítica, ningún otro hilo puede ingresar en ella simultáneamente. Previene la corrupción de datos.
* **Progreso:** Si la sección crítica está libre y varios hilos desean entrar, la decisión de quién lo hace no puede postergarse indefinidamente, evitando bloqueos innecesarios (*Deadlocks*).
* **Espera Acotada (Ausencia de Inanición):** Debe existir un límite garantizado en el número de veces que otros hilos pueden adelantarse antes de que se le permita el acceso a un hilo en espera (*Starvation*).

---

## 3. Soluciones Puramente de Software
Históricamente se demostró que es posible lograr exclusión mutua sin hardware especializado, utilizando algoritmos basados puramente en variables de software:

* **Algoritmo de Peterson (para 2 hilos):** Utiliza un arreglo de banderas de intención (`flag[]`) y una variable de turno (`turn`) para resolver el desempate mediante una solución matemática elegante que garantiza exclusión mutua, progreso y espera acotada. Sin embargo, su principal desventaja práctica es el costo de la **Espera Activa (*Busy-Wait*)**, donde los hilos consumen ciclos completos de CPU preguntando repetidamente en un bucle si ya pueden entrar.
* **La Panadería de Lamport (para $N$ hilos):** Escala el control de acceso a un número arbitrario de hilos simulando el sistema de un dispensador de tiquetes de una panadería, donde cada hilo toma un número secuencial único y se atiende en orden estricto, desempatando por el ID del hilo en caso de colisión.

---

## 4. El Giro de Trama: La Realidad de las CPUs Modernas
Los algoritmos clásicos de software puro (Peterson y Lamport) asumen una premisa que las arquitecturas modernas de hardware rompen: **el orden secuencial estricto de las instrucciones**.

Para maximizar el rendimiento, las CPUs actuales (x86/ARM) y sus jerarquías de caché emplean técnicas de **ejecución reordenada** y búferes de escritura (*Store Buffers*). Debido a esto, una instrucción de escritura en una variable de control puede quedarse atrapada temporalmente en la caché privada de un núcleo mientras otra avanza, haciendo que los algoritmos basados en software puro fallen físicamente a pesar de ser matemáticamente perfectos.

Para solventar esto, la ingeniería moderna evolved delegando la sincronización directamente al silicio mediante **Barreras de Memoria (*Memory Fences*)** e **Instrucciones Atómicas de Hardware** (como `test_and_set` o *Compare-And-Swap* - CAS), sentando las bases físicas para la construcción de los Semáforos y Mutexes utilizados en los sistemas operativos actuales.

---

## 5. Conclusión
El estudio de la sección crítica demuestra que la corrección lógica en papel (software) es insuficiente si se ignora el comportamiento físico del hardware subyacente. Comprender cómo operan las condiciones de carrera y la evolución desde los algoritmos de exclusión mutua hasta las instrucciones atómicas es indispensable para el diseño seguro de software concurrente de alto rendimiento.
