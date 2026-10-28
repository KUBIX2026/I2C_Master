# Especificación y restricciones del Sistema: Periférico I2C_Master.

---

## 1. Introducción y Propósito del Sistema
El presente repositorio implementa un bloque IP de hardware reutilizable correspondiente a un controlador maestro **$I^2C$ (*Inter-Integrated Circuit*)**, diseñado para integrarse como un periférico mapeado en memoria dentro de un System-on-Chip (SoC) basado en procesador (siguiendo la estructura de referencia de periféricos como `perip_blink.v`). 

Su función principal dentro del sistema general del videojuego es actuar como un puente de comunicación síncrona de alta confiabilidad para **almacenar y recuperar de forma no volátil las puntuaciones (*Scores*) de los jugadores en una memoria EEPROM externa** (ej. serie 24LCxx conectada en la dirección base `0x50`), liberando a la CPU de la gestión temporal bit a bit del protocolo.

---

## 2. Especificación Técnica y Restricciones

### 2.1. Definición de Interfaces, Entradas y Salidas (Pinout)
El periférico divide su interfaz en dos dominios: la **Interfaz del Bus del Sistema (Lado CPU)** y la **Interfaz Física del Bus $I^2C$ (Lado Externo)**.

| Señal | Dirección | Ancho | Dominio / Tipo | Descripción Funcional |
| :--- | :---: | :---: | :---: | :--- |
| **Interfaz del Bus del Procesador (SoC)** | | | | |
| `clk` | `input` | 1 bit | Reloj Sistema | Reloj principal del SoC (ej. $25\text{ MHz}$, $50\text{ MHz}$ o $100\text{ MHz}$) que sincroniza la FSM y el divisor de frecuencia. |
| `rst_n` | `input` | 1 bit | Reset (Activo Bajo) | Reinicio global asíncrono/síncrono; coloca los registros en su valor por defecto y libera el bus $I^2C$ (`1'bZ`). |
| `cs` | `input` | 1 bit | Control Bus | *Chip Select*: habilita el acceso al banco de registros del periférico cuando su dirección base es decodificada. |
| `wr` | `input` | 1 bit | Control Bus | Habilitación de escritura desde la CPU hacia el registro seleccionado por `addr`. |
| `rd` | `input` | 1 bit | Control Bus | Habilitación de lectura desde el registro seleccionado por `addr` hacia la CPU. |
| `addr` | `input` | 5 bits | Dirección Bus | Bus de direcciones local (`0x00` a `0x14`) para seleccionar uno de los 6 registros internos de 32 bits. |
| `wdata` | `input` | 32 bits | Datos Bus | Palabra de datos de 32 bits escrita por el procesador en los registros de configuración, dirección o transmisión. |
| `rdata` | `output` | 32 bits | Datos Bus | Palabra de datos de 32 bits leída por el procesador (datos recibidos desde la EEPROM o banderas de estado). |
| **Interfaz Física del Bus $I^2C$** | | | | |
| `sda` | `inout` | 1 bit | *Open-Drain* | Línea bidireccional de datos serie (*Serial Data*). Opera únicamente entre `1'b0` y alta impedancia (`1'bZ`). |
| `scl` | `inout` | 1 bit | *Open-Drain* | Línea bidireccional de reloj serie (*Serial Clock*). Generada por el maestro con soporte de *Clock Stretching*. |

### 2.2. Mapa de Memoria (*Memory Map*)
El periférico se mapea en el espacio de direcciones de 32 bits del procesador mediante 6 registros alineados a palabras de 4 bytes (`0x00` a `0x14`):

| Offset | Nombre del Registro | Acceso | Valor Reset | Descripción de Campos de Bits `[31:0]` |
| :---: | :--- | :---: | :---: | :--- |
| `0x00` | `I2C_CTRL` | R/W | `0x0000_0000` | **`[0] EN`**: Habilita el núcleo $I^2C$.<br>**`[1] START_CMD`**: Dispara condición `START` / `Repeated START` y transmisión de dirección.<br>**`[2] STOP_CMD`**: Solicita condición `STOP` al finalizar el byte actual.<br>**`[3] ACK_EN`**: Respuesta del maestro en lectura (`0` = enviar ACK para continuar leyendo; `1` = enviar NACK en el último byte).<br>**`[31:4]`**: Reservados (`0`). |
| `0x04` | `I2C_ADDR` | R/W | `0x0000_0000` | **`[0] RW`**: Operación (`0` = Escritura hacia esclavo, `1` = Lectura desde esclavo).<br>**`[7:1] SLAVE_ADDR`**: Dirección de 7 bits del esclavo destino (ej. `7'b1010000` = `0x50` para EEPROM).<br>**`[31:8]`**: Reservados (`0`). |
| `0x08` | `I2C_TXDATA` | R/W | `0x0000_0000` | **`[7:0] TX_BYTE`**: Byte de datos o dirección interna de memoria EEPROM a transmitir por `SDA` (orden MSB-first).<br>**`[31:8]`**: Reservados (`0`). |
| `0x0C` | `I2C_RXDATA` | RO | `0x0000_0000` | **`[7:0] RX_BYTE`**: Último byte completo recibido desde el esclavo en una operación de lectura.<br>**`[31:8]`**: Reservados (`0`). |
| `0x10` | `I2C_STATUS` | RO / W1C | `0x0000_0000` | **`[0] BUSY`** (RO): `1` mientras el bus está ocupado entre `START` y `STOP`.<br>**`[1] DONE`** (RO/W1C): `1` al finalizar la transferencia de un byte o condición de parada.<br>**`[2] ACK_ERR`** (RO/W1C): `1` si el esclavo responde con `NACK` (`SDA = 1`) en el 9.º pulso.<br>**`[3] ARB_LOST`** (RO/W1C): `1` si se detecta pérdida de arbitraje en el bus.<br>**`[31:4]`**: Reservados (`0`). |
| `0x14` | `I2C_PRESCALER` | R/W | `0x0000_007C` | **`[15:0] PRESCALE_VAL`**: Divisor de frecuencia para generar las 4 sub-fases por período de `SCL` ($\text{PRESCALER} = \frac{f_{CLK}}{4 \cdot f_{SCL}} - 1$).<br>**`[31:16]`**: Reservados (`0`). |

### 2.3. Restricciones Técnicas y Capa Física
1. **Topología *Open-Drain* (Drenador Abierto):** Queda estrictamente prohibido inyectar un nivel alto activo (`1'b1` *push-pull*) en los pines `sda` y `scl` para prevenir cortocircuitos cuando un esclavo fuerza un `0` lógico. Las salidas conmutan exclusivamente entre `1'b0` y alta impedancia (`1'bZ`), requiriendo resistencias de *pull-up* externas ($R_p$ entre $2.2\text{ k}\Omega$ y $4.7\text{ k}\Omega$ a $3.3\text{ V}$) con capacitancia de bus $C_{bus} \le 400\text{ pF}$.
2. **Regla de Validez de Datos:** `SDA` solo puede cambiar de valor mientras `SCL = 0` (en la sub-fase $Q_0$) y debe permanecer estrictamente estable durante todo el pulso alto `SCL = 1` ($Q_2$ y $Q_3$), excepto en las condiciones de control `START`, `Repeated START` y `STOP`.
3. **Sincronización Anti-Metaestabilidad:** Las entradas físicas `sda_in` y `scl_in` pasan obligatoriamente por un sincronizador de doble Flip-Flop y un filtro de picos de ruido (*spike filter* de $50\text{ ns}$) antes de ser evaluadas por la lógica secuencial.
4. **Restricción Temporal de EEPROM:** Tras una operación de escritura de puntaje y la condición `STOP`, la memoria EEPROM requiere un tiempo interno de ciclo de escritura de página no volátil ($t_{WR} \le 5\text{ ms}$), lapso durante el cual no emite `ACK` a nuevas órdenes.

### 2.4. Manejo de Errores y Recuperación
* **Fallo de Reconocimiento (`ACK_ERR`):** Si en el 9.º pulso de `SCL` el maestro muestrea `sda_in == 1` (`NACK`), activa `ACK_ERR = 1`, aborta la trama y ejecuta automáticamente un `STOP` para liberar el bus.
* **Pérdida de Arbitraje (`ARB_LOST`):** Si durante la transmisión de un bit `1` el maestro detecta `sda_in == 0`, activa `ARB_LOST = 1`, pone sus pines en alta impedancia (`1'bZ`) y retorna a `IDLE`.

---
