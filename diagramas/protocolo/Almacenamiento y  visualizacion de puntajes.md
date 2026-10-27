# Almacenamiento, Guardado y Recuperación de Puntuaciones en el Sistema I2C



## 1. Dónde se almacena la Puntuación (Mapa Físico y de Memoria)

El almacenamiento de las puntuaciones se divide en dos niveles jerárquicos:

### A. Almacenamiento No Volátil Permanente (Memoria Externa EEPROM $I^2C$)
* **Dispositivo Físico:** Una memoria EEPROM externa (típicamente modelos de la familia `24LCxx` como la `24LC256` de $32\text{ KB}$ o `24LC64` de $8\text{ KB}$) conectada al bus físico mediante los pines compartidos **SDA** y **SCL** del FPGA.
* **Direccionamiento del Esclavo:** La EEPROM responde a una dirección base fija definida por hardware en sus pines de configuración (por ejemplo, `0x50` en binario `1010000`).
* **Organización del Mapa de Memoria Interno de la EEPROM:**
  * Las puntuaciones de los juegos no se guardan de manera volátil (se pierden al apagar el FPGA), sino de forma estructurada en celdas de memoria de 8 bits (bytes).
  * Cada partida o récord requiere un bloque de bytes (por ejemplo, un puntaje de 16 bits o 32 bits requiere particionarse en 2 o 4 celdas consecutivas de la EEPROM, acompañadas opcionalmente de iniciales de tres letras del jugador y un byte de validación o *checksum*).

### B. Almacenamiento Temporal Volátil (Registros del Procesador y FPGA)
* **Registros Internos del Periférico `perip_i2c_master`:**
  * `I2C_TXDATA` (`0x08`): Almacena temporalmente el byte de puntuación que está siendo enviado al bus.
  * `I2C_RXDATA` (`0x0C`): Contiene el último byte de puntuación recuperado desde la EEPROM durante una operación de lectura.
* **Memoria RAM del Procesador (SoC):** El software en C mantiene una variable global (ej. `uint16_t current_score`) que almacena la puntuación en curso mientras el juego está activo antes de ser volcada de manera permanente a la EEPROM.

---

## 2. Cómo se Guarda la Puntuación (Proceso de Escritura)

Cuando un jugador finaliza su partida y se alcanza un nuevo récord, el procesador ejecuta una secuencia de escritura en la memoria EEPROM a través de los registros del controlador maestro $I^2C$:

1. **Preparación del Software (C):**
   * El programa en C formatea el puntaje (ej. $1250$ decimal $\rightarrow$ `0x04` y `0xE2` si se divide en dos bytes).
   * El software escribe la dirección de memoria interna de la EEPROM donde se desea guardar el puntaje (ej. `0x0010`) en el registro de dirección del maestro.
2. **Disparo de la Transacción por el Maestro (`I2C_Master`):**
   * **Fase START ($S$):** El maestro toma el control del bus bajando SDA con SCL en alto.
   * **Envío de Dirección del Esclavo + Escritura (`0x50` + Bit $R/\overline{W} = 0$):** El maestro transmite el byte de control de la EEPROM. La EEPROM responde con un pulso **ACK (`0`)** en el 9.º ciclo de reloj.
   * **Envío de la Dirección de Memoria Interna (Word Address):** El maestro transmite la posición de la EEPROM (ej. `0x10`) donde se almacenará el puntaje. La EEPROM responde con **ACK**.
3. **Escritura de los Bytes de Puntuación (Data Bytes):**
   * El procesador escribe el byte alto del puntaje en `I2C_TXDATA`, el hardware lo desplaza bit a bit por SDA y espera el **ACK** de la EEPROM.
   * Se repite el proceso para el byte bajo del puntaje.
4. **Condición de Parada (`STOP` - $P$):**
   * El maestro genera una condición de STOP subiendo SDA con SCL en alto, liberando el bus.
   * La EEPROM inicia internamente su ciclo de escritura física no volátil (*Write Cycle Time*, típicamente de 5 ms).

---

## 3. Cómo se Recupera la Puntuación para Mostrarla en Pantalla (Proceso de Lectura)

Para mostrar las puntuaciones más altas (*High Scores*) en una pantalla (por ejemplo, una pantalla LCD por interfaz SPI/I2C o un controlador de video VGA implementado en el propio FPGA):

1. **Solicitud de Lectura (Transacción Combinada con *Repeated Start*):**
   * El procesador ordena leer desde una dirección específica de la EEPROM.
   * El maestro $I^2C$ ejecuta un comando `START`, envía la dirección de la EEPROM con bit de escritura (`0`), y transmite la dirección interna del registro de puntajes que desea consultar.
   * En lugar de cerrar la comunicación con un STOP, el maestro emite un **Repeated START ($Sr$)** para conservar la posesión del bus.
   * El maestro envía nuevamente la dirección de la EEPROM pero con el bit de lectura activado ($R/\overline{W} = 1$).
2. **Recepción de Datos (Master Read):**
   * La EEPROM toma el control de la línea SDA y comienza a transmitir los bytes de la puntuación almacenada.
   * El hardware del maestro captura cada bit en cada pulso alto de SCL y reconstruye el byte completo dentro de `I2C_RXDATA`.
   * Para el último byte que se desea leer, el maestro emite un **NACK** explícito seguido de una condición de **STOP ($P$)**, indicándole a la EEPROM que detenga la transmisión.
3. **Procesamiento y Despliegue en Pantalla:**
   * El procesador lee el registro `I2C_RXDATA`, recombina los bytes recuperados (ej. `(byte_alto << 8) | byte_bajo`) para obtener el valor numérico original de la puntuación.
   * El software traduce este número entero a caracteres ASCII o patrones gráficos de texto.
   * Finalmente, estos datos son enviados al periférico de pantalla (controlador VGA o driver de display) para actualizar visualmente la tabla de récords en tiempo real.

---

## 4. Resumen del Flujo de Datos

```
[ Fin de Partida / Récord ] 
       │
       ▼
[ Procesador (SoC) ] ──(Escribe registros: CTRL, ADDR, TXDATA)──► [ Periférico I2C_Master ]
                                                                      │
                                                             (Protocolo I2C / SDA-SCL)
                                                                      ▼
                                                       [ Memoria EEPROM Externa (No Volátil) ]
                                                                      │
                                                           (Lectura vía Repeated START)
                                                                      ▼
[ Pantalla / Display ] ◄──(Recombinación de bytes RXDATA)────── [ Periférico I2C_Master ]
