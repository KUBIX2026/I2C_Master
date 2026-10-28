## 1. Modelo y Arquitectura del Sistema (Criterio C2)

### 1.1. Diagrama de flujo de alto nivel. 

## 1.2 Diagrama de bloques.
* **Diagrama de bloques general:** 
<img width="736" height="477" alt="Captura desde 2026-10-05 05-35-00" src="https://github.com/user-attachments/assets/8f54781b-395f-4de5-840c-a57e91b409fa" />

**Figura 1. Diagrama de bloques general protocolo I2C.**

Example of Single-Master Transmit/Receive Using I2C Bus Interface. http://www.renesas.com/. abril 2010

* **MASTER:** Decide qué operación I²C realizar.
* <img width="412" height="651" alt="Diagrama sin título(1)" src="https://github.com/user-attachments/assets/0a72a99a-ca79-475a-afae-48b39216bdbe" />

**Figura 1.2. Diagrama de bloques MASTER protocolo I2C.**

* **CONTROL UNIT:** Decide qué operación debe realizarse y en qué momento, mediante una máquina de estados (FSM).
<img width="412" height="651" alt="CONTROLBLOCK" src="https://github.com/user-attachments/assets/8fa5d96a-8d2f-49b9-836d-65e02f78a2f3" />

**Figura 1.2.1 Diagrama de bloques CONTROL UNIT protocolo I2C.**

* **DATA PATH:** Contiene los registros, contadores, multiplexores y demás circuitos que manipulan, transmiten y reciben los datos durante la comunicación.
<img width="412" height="561" alt="pathBLOCK" src="https://github.com/user-attachments/assets/69c5b2e3-d7c3-4dc4-bfd5-da1c5bdddc2e" />

**Figura 1.3 Diagrama de bloques DATA PATH protocolo I2C.**

* **MASTER FINAL:** Controla y coordina la comunicación I²C con el dispositivo esclavo, generando las señales SCL y SDA necesarias para leer o escribir datos.
* <img width="528" height="1062" alt="GENERALBLOCK" src="https://github.com/user-attachments/assets/b1c6b2d0-25a4-491f-b3ad-0cb23963d355" />
**Figura 2 Diagrama de bloques I2C MASTER diseño protocolo I2C.**

## 2. Almacenamiento, Guardado y Recuperación de Puntuaciones (*Scores*)

### 2.1. ¿Dónde se almacena el Score?
El puntaje del juego posee dos niveles de almacenamiento:
1. **Volátil (RAM del Procesador):** Durante el transcurso de la partida, la puntuación se actualiza en tiempo real en registros y variables alojadas en la memoria RAM interna del procesador SoC.
2. **No Volátil (Memoria EEPROM Externa 24LCxx):** Al finalizar la partida, el puntaje es transferido mediante el periférico `perip_i2c_master` hacia una memoria EEPROM externa conectada en la dirección `0x50`, garantizando que los récords persistan aunque el sistema se apague.

### 4.2. ¿Cómo se guarda el Score? (Secuencia de Escritura)
Cuando el juego termina, el firmware en C invoca al driver `i2c.c` para realizar una transacción de escritura por ráfaga (*Byte/Page Write*):
1. **Disparo START:** La CPU configura `I2C_ADDR = 0x50` con bit $R/\bar{W} = 0$ (Escritura) y activa `START_CMD = 1`.
2. **Envío de Dirección de Memoria Interna:** Escribe en `I2C_TXDATA` la dirección de celda dentro de la EEPROM donde se guardará el puntaje y espera `DONE = 1`.
3. **Envío de Bytes del Puntaje:** Envía secuencialmente los bytes que componen la variable del *Score* (ej. 2 o 4 bytes) depositándolos en `I2C_TXDATA`.
4. **Condición STOP y Espera de Grabación:** El firmware ordena `STOP_CMD = 1`. Tras el `STOP`, el software aguarda el tiempo de ciclo interno de escritura de la EEPROM ($t_{WR} \le 5\text{ ms}$).

### 2.3. ¿Cómo se recupera el Score para mostrarlo en pantalla?
Para mostrar la tabla de mejores puntajes en la interfaz gráfica, el procesador recupera la información ejecutando una **Transacción Combinada con Repeated START**:
1. **Fase de Apuntamiento (Escritura de Dirección):** El maestro emite `START`, direcciona la EEPROM con $R/\bar{W} = 0$ y envía el byte de dirección de celda del puntaje.
2. **Cambio de Modo (Repeated START):** Sin liberar el bus, el maestro emite un `Repeated START` (`START_CMD = 1`), direcciona nuevamente la EEPROM (`0x50`) pero con bit $R/\bar{W} = 1$ (Lectura).
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
* **Protocolo $I^2C$ de 2 Hilos (Elegida) vs. SPI:** Optimiza el uso de pines en la FPGA a solo 2 líneas (`SDA`, `SCL`) y provee confirmación nativa de recepción (`ACK/NACK`) ideal para memorias de almacenamiento de puntajes.

---

## 4. Criterios de Aceptación Verificables (Checklist para el Jueves)
* [ ] **CA-01:** Las formas de onda en simulación (`GTKWave`) confirman operación puramente *Open-Drain* en `SDA` y `SCL`.
* [ ] **CA-02:** Con el `PRESCALER` configurado, la frecuencia de `SCL` se estabiliza exactamente en $100\text{ kHz} \pm 1\%$.
* [ ] **CA-03:** El módulo detecta esclavos ausentes levantando la bandera `ACK_ERR = 1` y ejecutando `STOP`.
* [ ] **CA-04:** Se logra escribir y leer exitosamente una puntuación de prueba en la EEPROM mediante transacción combinada.
