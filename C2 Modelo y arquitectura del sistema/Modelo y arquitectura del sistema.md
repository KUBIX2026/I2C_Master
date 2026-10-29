## 1. Modelo y Arquitectura del Sistema (Criterio C2)

### 1.1 Diagrama de flujo de alto nivel. 
<img width="577" height="1202" alt="Diagrama flujo-I2C_Master_Flowchart drawio (3)" src="https://github.com/user-attachments/assets/b1c9b4f8-bcb7-4731-a5c5-e9f10ac89890" />

**Figura 1. Diagrama de flujo general protocolo I2C.**

### 1.2 Diagrama de bloques general.

<img width="736" height="477" alt="Captura desde 2026-10-05 05-35-00" src="https://github.com/user-attachments/assets/8f54781b-395f-4de5-840c-a57e91b409fa" />

* **Diagrama de bloques concreto (I²C Master propuesto)**

<img width="539" height="181" alt="image" src="https://github.com/user-attachments/assets/15919172-0095-4d7e-a9da-753ec793fbbb" />

**Figura 2. Diagrama de bloques general protocolo I2C. (a) Diagrama general de referencia. (b) Diagrama concreto del I²C Master propuesto.**

Fuente de (a): Example of Single-Master Transmit/Receive Using I2C Bus Interface. http://www.renesas.com/. abril 2010

* **MASTER:** Decide qué operación I²C realizar.
<img width="373" height="164" alt="image" src="https://github.com/user-attachments/assets/472ec292-c980-4336-9d71-af927fd303a6" />
<img width="461" height="135" alt="image" src="https://github.com/user-attachments/assets/b918af84-22fc-4497-9fb2-f53791ad7282" />



**Figura 2.1 Diagrama de bloques MASTER protocolo I2C.**

* **CONTROL UNIT:** Decide qué operación debe realizarse y en qué momento, mediante una máquina de estados (FSM).
<img width="294" height="170" alt="image" src="https://github.com/user-attachments/assets/d9b13c6e-401a-4aaf-bce4-b23402c3c9ca" />
<img width="464" height="209" alt="image" src="https://github.com/user-attachments/assets/8fcd07e4-3a7d-4506-a332-128f919d362e" />


**Figura 2.2 Diagrama de bloques CONTROL UNIT protocolo I2C.**

* **DATA PATH:** Contiene los registros, contadores, multiplexores y demás circuitos que manipulan, transmiten y reciben los datos durante la comunicación.
<img width="321" height="115" alt="image" src="https://github.com/user-attachments/assets/27378702-a633-4b33-93e6-909a28970fd3" />
<img width="462" height="149" alt="image" src="https://github.com/user-attachments/assets/c045927f-5dcf-45ae-89f0-fb5436cfe61e" />


**Figura 2.3 Diagrama de bloques DATA PATH protocolo I2C.**

* **MASTER FINAL:** Controla y coordina la comunicación I²C con el dispositivo esclavo, generando las señales SCL y SDA necesarias para leer o escribir datos.
 <img width="303" height="108" alt="image" src="https://github.com/user-attachments/assets/32d9a588-b632-4ab2-8820-34213ca57dd9" />
 <img width="459" height="122" alt="image" src="https://github.com/user-attachments/assets/8a8b9bf5-62cf-4f4e-b1b9-cb183dd9d48d" />
 
**Figura 2.4 Diagrama de bloques MASTER FINAL protocolo I2C.**

* **Generador de reloj:** Genera la señal de temporización `clk_i2c` y el pulso `tick` a partir del reloj del sistema y del valor de `prescaler`. Este valor define la frecuencia del bus (100 kHz / 400 kHz) y sincroniza al Control Unit, al Data Path y al Master Final.
<img width="278" height="87" alt="image" src="https://github.com/user-attachments/assets/373e3b0a-0782-4d77-a747-e7788101e62a" />
<img width="439" height="91" alt="image" src="https://github.com/user-attachments/assets/4843e41c-3b76-4405-a80b-a637a912c5b0" />

  
**Figura 2.5 Diagrama de bloques del Generador de reloj.**

* **Registros de control y estado:** Almacenan la configuración de la operación y el estado del controlador. Permiten escribir y leer registros (`wr_en`, `rd_en`) y consultar el estado (`status[2:0]`), por lo que sirven de interfaz entre el sistema y el I²C Master.
<img width="401" height="124" alt="image" src="https://github.com/user-attachments/assets/92cad669-fc0b-4a67-912a-b6d4786f97f7" />
<img width="556" height="146" alt="image" src="https://github.com/user-attachments/assets/022c829b-47e1-4253-86a8-a95fbf55d5fb" />
  
**Figura 2.6 Diagrama de bloques de los Registros de control y estado.**

* **Secuencia de una transferencia I2C:** Muestra el orden de pasos de una transferencia, Corresponde solo al protocolo, no a un bloque del hardware.
<img width="389" height="141" alt="image" src="https://github.com/user-attachments/assets/e899369d-7883-474b-980f-e3c7b37ea20a" />
<img width="556" height="142" alt="image" src="https://github.com/user-attachments/assets/13dd3ad2-fdfa-4533-9c19-a66827713123" />

  
**Figura 2.7 Secuencia de una transferencia I2C (solo protocolo).**


### 1.3 Diagrama de flujo operacional (Algoritmo de lectura y escritura del I²C Master).
<img width="1688" height="2532" alt="flujo_i2c_master_page-0001" src="https://github.com/user-attachments/assets/3a0f98bd-3aa9-4daa-beb9-7d399be56e6b" />


**Figura 3. Diagrama de flujo del algoritmo de transferencia del I²C Master.**

## 2. Almacenamiento, Guardado y Recuperación de Puntuaciones (*Scores*)

### 2.1. ¿Dónde se almacena el Score?
El puntaje del juego posee dos niveles de almacenamiento:
1. **Volátil (RAM del Procesador):** Durante el transcurso de la partida, la puntuación se actualiza en tiempo real en registros y variables alojadas en la memoria RAM interna del procesador SoC.
2. **No Volátil (Memoria EEPROM):** Al finalizar la partida, el puntaje es transferido mediante el periférico `perip_i2c_master` hacia una memoria EEPROM externa, garantizando que los récords persistan aunque el sistema se apague.
La disposición de los pines y su conexion se puede describir de la siguiente manera:
  

    
| Pin | Función |
|---|---|
| **SCL** | Reloj serie. Los datos **entran** por flanco de subida y **salen** por flanco de bajada. |
| **SDA** | Datos bidireccionales, de **drenador abierto**. Puede compartirse con otros dispositivos open-drain; requiere resistencia de pull-up externa. |
| **A2, A1, A0** | Dirección física del dispositivo, cableada a VCC o GND. Permiten hasta 8 chips de 1K/2K en el mismo bus. |
| **WP** | Protección de escritura. A **VCC** protege toda la matriz; a **GND** permite lectura y escritura normal. |
| **VCC / GND** | Alimentación y tierra. |

 2.1.1 Diagrama de bloques
 
 <img width="906" height="608" alt="image" src="https://github.com/user-attachments/assets/4dc8e9e6-9d14-43bd-8875-c7077f04ee2f" />

Fuente de (b): [Example of Single-Master Transmit/Receive Using I2C Bus Interface. http://www.renesas.com/. abril 2010](https://www.alldatasheet.com/datasheet-pdf/view/56236/ATMEL/AT24CXX.html)

El diagrama de bloques fue tomado del datasheet de un modelo de la familia de 224Cxx (Modelo AT24C01A/02/04/08A/16A), donde se ejemplifica el comportamiento externo del chip. 
vemos que el chip EEPROM recibe por SCL y SDA los comandos del maestro. La lógica de control interpreta el comando de acuerdo al seguimiento de la secuencia de las muestras, valida la dirección del dispositivo, maneja un contador interno de dirección y activa, según el caso, la ruta de **escritura**  o la ruta de **lectura**.

| Bloque | Función |
|---|---|
| **START/STOP logic** | Vigila SCL y SDA para detectar las condiciones de **START** y **STOP**, e informa a la lógica de control. |
| **Serial control logic** | Unidad de control (máquina de estados). Recibe SCL, SDA y WP, y genera las señales **LOAD**, **INC** y **EN**. Coordina todo el chip. |
| **Device address comparator** | Compara los 7 bits recibidos (`1010` + A2A1A0) con los pines físicos. Si coinciden produce **COMP** y el chip responde. También separa el bit **R/W**. |
| **Data word addr/counter** | Contador que guarda la dirección de la palabra de memoria. Se carga con **LOAD** y se incrementa con **INC**; es el **puntero autoincremental**. |
| **X DEC / Y DEC** | Decodificadores de **filas** y **columnas**; con la dirección del contador seleccionan la celda en la matriz. |
| **EEPROM + Data recovery** | Matriz de memoria y amplificadores de lectura que recuperan el dato almacenado. |
| **Serial MUX** | Selecciona el bit de la palabra leída que se envía hacia la salida. |
| **H.V. pump/timing** | Bomba de carga y temporización para programar celdas. La activa la señal **EN**. Es el origen del ciclo interno autotemporizado (t_WR ≤ 5 ms). |
| **D_OUT/ACK logic** | Maneja SDA en la salida: coloca los bits de lectura y genera el **ACK** (un cero en el noveno pulso). Actúa sobre un transistor a tierra, por eso SDA es de drenador abierto. |

<img width="417" height="505" alt="image" src="https://github.com/user-attachments/assets/271727fc-562a-47ce-9db4-3f754f9474d6" />


### 2.2. ¿Cómo se guarda el Score? (Secuencia de Escritura)
Cuando el juego termina, el firmware en C invoca al driver `i2c.c` para realizar una transacción de escritura por ráfaga (*Byte Write/Page Write*):

<img width="941" height="619" alt="image" src="https://github.com/user-attachments/assets/e30e214b-daa8-4b7f-8897-281087a25e0b" />


1. **Disparo START:** La CPU configura `I2C_ADDR = 0x50` con bit $R/\bar{W} = 0$ (Escritura) y activa `START_CMD = 1`.
2. **Envío de Dirección de Memoria Interna:** Escribe en `I2C_TXDATA` la dirección de celda dentro de la EEPROM donde se guardará el puntaje y espera `DONE = 1`.
3. **Envío de Bytes del Puntaje:** Envía secuencialmente los bytes que componen la variable del *Score* (ej. 2 o 4 bytes) depositándolos en `I2C_TXDATA`, mientras que en modo de operación Byte Write se escrobe un bite por operación requiriendo direcciones detalladas para cada uno, el modo de operación Page Write puede escribir varios Bytes en una sola operación.
4. La escriturs por página se inicia igual que La escritura por byte, pero el maestro no envía STOP tras el primer dato. Mientras la EEPROM responda ACK, el maestro puede enviar de 7 a 15 datos mas para páginas de 8 a 16 bytes.
5. **Condición STOP y Espera de Grabación:** El firmware ordena `STOP_CMD = 1`. Tras el `STOP`, el software aguarda el tiempo de ciclo interno de escritura de la EEPROM ($t_{WR} \le 5\text{ ms}$).

### 2.3. ¿Cómo se recupera el Score para mostrarlo en pantalla?
Para mostrar la tabla de mejores puntajes en la interfaz gráfica, el procesador recupera la información ejecutando una **Transacción Combinada con Repeated START**:

El proceso de lectira se puede dar con dos diferentes procesos o modos de operación:

<img width="914" height="565" alt="image" src="https://github.com/user-attachments/assets/2a257fa0-b007-4cfc-ab24-818758ddbf05" />


1. **Fase de Apuntamiento (Escritura de Dirección):** El maestro emite `START`, direcciona la EEPROM con $R/\bar{W} = 0$ y envía el byte de dirección de celda del puntaje.
2. **Cambio de Modo (Repeated START):** Sin liberar el bus, el maestro emite un `Repeated START` (`START_CMD = 1`), direcciona nuevamente la EEPROM (`0x50`) pero con bit $R/\bar{W} = 1$ (Lectura). Para leer una dirección específica hay que cargar primero el puntero con una escritura ficticia (dummy write), sin llegar a escribir ningún dato. Si bien los dos modos de operación inician igual, su diferencia clave es que mientras la lectura aleatoria permite leer un byte en una dirección especifica, la lectura secuencial permite leer bytes consecutivos a la direccción inicial.
3. **Lectura de Datos y Master NACK:** El hardware recibe los bytes del puntaje y los entrega a la CPU a través de `I2C_RXDATA`. Para el último byte leído, el maestro responde con un `NACK` (`ACK_EN = 1`) y emite la condición `STOP`.
4. **Despliegue en Pantalla:** La CPU toma el valor recuperado de `I2C_RXDATA` y lo envía al controlador gráfico para renderizar el puntaje en la pantalla.

---

## 3. Partición Hardware-Software y Análisis de Alternativas

### 3.1. Partición Hardware-Software (*HW/SW Partitioning*)
| Subsistema | Responsabilidad Asignada | Justificación Técnica |
| :--- | :---: | :--- |
| **Hardware** (`perip_i2c_master.v` + `i2c_master_core.v`) | Generación de reloj `SCL` y cuadrantes $Q_0\text{–}Q_3$, serialización/deserialización MSB-first de 8 bits, conteo del pulso 9 (`ACK/NACK`), sincronización y control *Open-Drain*. | Requiere precisión determinista a nivel de microsegundos sin bloquear el ciclo de instrucciones de la CPU. |
| **Software** (`i2c.c`, `i2c.h` y Aplicación) | Configuración del `PRESCALER`, segmentación del *Score* en bytes, direccionamiento de celdas de la EEPROM, control de tiempos $t_{WR}$ y lógica de renderizado en pantalla. | Otorga flexibilidad para modificar estructuras de datos o protocolos de visualización sin resintetizar la FPGA. |

### 3.2. Análisis de Alternativas Arquitectónicas
* **Periférico Hardware RTL (Elegida) vs. *Bit-Banging* por Software:** Descarga a la CPU de retardos bloqueantes de $\sim 90\text{ }\mu\text{s}$ por byte, garantizando márgenes temporales estrictos para el bus $I^2C$.
* **Diseño Síncrono con Habilitación de 4 Fases `tick_4x` (Elegida) vs. Relojes Múltiples:** Opera 100% bajo un único dominio de reloj (`clk`), evitando problemas de cruce de dominios (*CDC*) y garantizando estabilidad en el muestreo.
* **Protocolo $I^2C$ de 2 Hilos (Elegida) vs. SPI:** Optimiza el uso de pines en la FPGA a solo 2 líneas (`SDA`, `SCL`) y provee confirmación nativa de recepción (`ACK/NACK`) igual de ideal para memorias de almacenamiento de puntajes.

---
