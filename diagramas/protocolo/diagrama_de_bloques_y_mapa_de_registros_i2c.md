# Arquitectura de Hardware y Mapa de Registros — `perip_i2c_master`

Este documento especifica la arquitectura en bloques a nivel RTL (*Register Transfer Level*) y el **Mapa de Registros de Memoria (*Memory Map*)** de $32\text{ bits}$ del periférico maestro $I^2C$ (`perip_i2c_master.v`), diseñado para conectarse como periférico mapeado en memoria al bus del procesador en el SoC (siguiendo la interfaz estándar de `perip_blink.v`).

---

## 1. Diagrama de Bloques del Periférico (`perip_i2c_master`)

El diseño se divide en dos capas principales:
1. **Envoltura de Bus (`perip_i2c_master.v`):** Decodifica las direcciones del procesador (`0x00` a `0x14`), gestiona las escrituras/lecturas del banco de registros de $32\text{ bits}$ y conecta las señales de control/estado con el núcleo.
2. **Núcleo del Protocolo (`i2c_master_core.v`):** Contiene el divisor de frecuencia de $4$ fases ($Q_0, Q_1, Q_2, Q_3$), la máquina de estados finitos (FSM), el registro de desplazamiento (*Shift Register* Tx/Rx), el contador de bits y la lógica *Open-Drain* con sincronizadores de entrada.

```mermaid
flowchart LR
    subgraph CPU_BUS ["Bus del Procesador (SoC)"]
        CLK["clk / rst"]
        CS["cs (Chip Select)"]
        WR_RD["wr / rd"]
        ADDR["addr[4:0]"]
        DIN["d_in[31:0]"]
        DOUT["d_out[31:0]"]
    end

    subgraph PERIP_WRAPPER ["Envoltura de Bus (perip_i2c_master.v)"]
        DEC["Decodificador de Direcciones\n y Banco de Registros"]
        REG_CTRL["0x00: I2C_CTRL"]
        REG_ADDR["0x04: I2C_ADDR"]
        REG_TX["0x08: I2C_TXDATA"]
        REG_RX["0x0C: I2C_RXDATA"]
        REG_STAT["0x10: I2C_STATUS"]
        REG_PRES["0x14: I2C_PRESCALER"]
    end

    subgraph I2C_CORE ["Núcleo RTL (i2c_master_core.v)"]
        PRESCALER["Generador de Reloj\n4 Fases (Q0..Q3)\n+ Clock Stretching"]
        FSM["Unidad de Control\n(FSM Protocolo I2C)"]
        SR["Shift Register 8-bit\n(Tx / Rx) + Bit Counter"]
        SYNC["Sincronizador 2-FF\ny Control Open-Drain"]
    end

    subgraph PADS ["Bus Físico I2C"]
        SCL_PIN(("SCL (inout)"))
        SDA_PIN(("SDA (inout)"))
    end

    CLK --> DEC
    CS --> DEC
    WR_RD --> DEC
    ADDR --> DEC
    DIN --> DEC
    DEC --> DOUT

    DEC <--> REG_CTRL
    DEC <--> REG_ADDR
    DEC <--> REG_TX
    DEC <-- Lectura --- REG_RX
    DEC <-- Lectura / W1C --- REG_STAT
    DEC <--> REG_PRES

    REG_PRES --> PRESCALER
    REG_CTRL --> FSM
    REG_ADDR --> SR
    REG_TX --> SR
    SR --> REG_RX
    FSM --> REG_STAT
    PRESCALER --> FSM
    FSM <--> SR
    FSM --> SYNC
    SR <--> SYNC

    SYNC <--> SCL_PIN
    SYNC <--> SDA_PIN
```

---

## 2. Interfaz de Puertos del Módulo Top (`perip_i2c_master.v`)

| Puerto | Dirección | Ancho | Descripción Funcional |
| :--- | :---: | :---: | :--- |
| `clk` | `input` | $1\text{ bit}$ | Reloj principal del sistema/procesador ($f_{\text{CLK}}$). |
| `rst` | `input` | $1\text{ bit}$ | Reinicio síncrono/asíncrono del sistema (activo en alto `1'b1`). |
| `cs` | `input` | $1\text{ bit}$ | *Chip Select*: habilita el acceso al periférico cuando `cs == 1'b1`. |
| `wr` | `input` | $1\text{ bit}$ | Habilitación de escritura desde la CPU hacia el registro seleccionado. |
| `rd` | `input` | $1\text{ bit}$ | Habilitación de lectura desde el registro seleccionado hacia la CPU. |
| `addr` | `input` | $5\text{ bits}$ | Offset de dirección en bytes (`0x00` a `0x14`) alineado a palabras de $32\text{ bits}$. |
| `d_in` | `input` | $32\text{ bits}$ | Bus de datos de entrada (escritura del procesador al periférico). |
| `d_out` | `output` | $32\text{ bits}$ | Bus de datos de salida (lectura del procesador desde el periférico). |
| `irq` | `output` | $1\text{ bit}$ | Línea opcional de solicitud de interrupción hacia el procesador. |
| `sda` | `inout` | $1\text{ bit}$ | Línea bidireccional de datos serie $I^2C$ (Drenador Abierto: `1'b0` o `1'bZ`). |
| `scl` | `inout` | $1\text{ bit}$ | Línea bidireccional de reloj serie $I^2C$ (Drenador Abierto: `1'b0` o `1'bZ`). |

---

## 3. Mapa de Registros de Memoria (*Memory Map*) — Resumen General

Todos los registros están alineados a fronteras de $32\text{ bits}$ ($4\text{ bytes}$) respecto a la dirección base del periférico (`I2C_BASE_ADDR`):

| Offset | Nombre del Registro | Acrónimo | Acceso | Valor de Reset | Descripción Breve |
| :---: | :--- | :--- | :---: | :---: | :--- |
| `0x00` | Registro de Control y Comandos | `I2C_CTRL` | `R/W` | `0x0000_0000` | Habilitación del núcleo, disparo de condiciones $S$, $Sr$, $P$, transferencia de bytes, control de ACK e interrupción. |
| `0x04` | Registro de Dirección Esclavo | `I2C_ADDR` | `R/W` | `0x0000_0000` | Dirección de $7\text{ bits}$ del esclavo (`[7:1]`) y bit de operación $R/\overline{W}$ (`[0]`). |
| `0x08` | Registro de Datos a Transmitir | `I2C_TXDATA` | `R/W` | `0x0000_0000` | Byte de datos (`[7:0]`) que el maestro enviará al esclavo en modo escritura. |
| `0x0C` | Registro de Datos Recibidos | `I2C_RXDATA` | `RO` | `0x0000_0000` | Último byte de datos (`[7:0]`) capturado desde el esclavo en modo lectura. |
| `0x10` | Registro de Estado del Bus y FSM | `I2C_STATUS` | `RO / W1C` | `0x0000_00C0` | Banderas de operación (`BUSY`, `DONE`, `ACK_ERR`, `BUS_BUSY`, `ARB_LOST`) y monitoreo de pines. |
| `0x14` | Registro Divisor de Reloj | `I2C_PRESCALER` | `R/W` | `0x0000_003E` | Divisor de $16\text{ bits}$ (`[15:0]`) para configurar la frecuencia de `SCL` ($100\text{ kHz}$, $400\text{ kHz}$, $1\text{ MHz}$). |

> **Convenciones de Acceso:**
> * **`R/W` (*Read/Write*):** El software puede leer y escribir el bit libremente.
> * **`RO` (*Read-Only*):** Solo lectura. Las escrituras por parte de la CPU son ignoradas.
> * **`R/W (AC)` (*Auto-Clear*):** El software escribe `1` para disparar un comando; el hardware lo limpia automáticamente a `0` una vez que la FSM acepta o completa la orden.
> * **`W1C` (*Write-1-to-Clear*):** El hardware pone el bit en `1` cuando ocurre el evento; el software escribe `1` en esa posición para limpiar la bandera a `0`.

---

## 4. Especificación Detallada a Nivel de Bit (`[31:0]`)

### 4.1. `0x00` — `I2C_CTRL` (Registro de Control y Comandos)

Controla la habilitación del periférico y las órdenes enviadas a la Máquina de Estados Finitos (FSM).

* **Distribución de Bits:**
  * `[31:8]`: `RESERVED`
  * `[7]`: `SWRST`
  * `[6]`: `IRQ_EN`
  * `[5]`: `ACK_M`
  * `[4]`: `STOP_CMD`
  * `[3]`: `REP_START_CMD`
  * `[2]`: `DATA_CMD`
  * `[1]`: `START_CMD`
  * `[0]`: `CORE_EN`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:8]` | `RESERVED` | `RO` | `24'h000000` | **Reservado:** Siempre retorna `0` al leerse. Escribir `0` para compatibilidad futura. |
| `[7]` | `SWRST` | `R/W (AC)` | `1'b0` | **Reinicio por Software (*Software Reset*):** Escribir `1` fuerza a la FSM al estado `IDLE`, limpia las banderas de estado (`I2C_STATUS`) y libera las líneas `SDA` y `SCL` a alta impedancia (`1'bZ`). Se limpia automáticamente en el siguiente ciclo de reloj. |
| `[6]` | `IRQ_EN` | `R/W` | `1'b0` | **Habilitación de Interrupción:**<br>`0`: Salida `irq` deshabilitada.<br>`1`: Activa la salida `irq` cuando `DONE == 1` o `ACK_ERR == 1`. |
| `[5]` | `ACK_M` | `R/W` | `1'b0` | **Bit de Reconocimiento del Maestro (Modo Lectura):** Define qué bit enviará el maestro en el $9.^\circ$ pulso de `SCL` tras recibir un byte en `I2C_RXDATA`:<br>`0`: Maestro envía **ACK** (`SDA = 0`), indicando al esclavo que desea seguir leyendo más bytes.<br>`1`: Maestro envía **NACK** (`SDA = 1`), indicando al esclavo que fue el último byte leído y que debe liberar `SDA`. |
| `[4]` | `STOP_CMD` | `R/W (AC)` | `1'b0` | **Comando de Parada ($P$):**<br>`1`: Ordena a la FSM generar la condición **STOP** ($\text{SDA}: 0 \rightarrow 1$ con $\text{SCL} = 1$) al finalizar el comando en curso o de forma inmediata si el bus está retenido en espera. Se autolimpia a `0`. |
| `[3]` | `REP_START_CMD` | `R/W (AC)` | `1'b0` | **Comando de Inicio Repetido ($Sr$):**<br>`1`: Ordena generar una condición **Repeated START** ($\text{SDA}: 1 \rightarrow 0$ con $\text{SCL} = 1$) seguida del envío del registro `I2C_ADDR` actualizado, permitiendo conmutar de escritura a lectura sin liberar el bus. Se autolimpia a `0`. |
| `[2]` | `DATA_CMD` | `R/W (AC)` | `1'b0` | **Comando de Transferencia de Byte de Datos:**<br>`1`: Si `RW_BIT == 0` (Escritura), transmite los $8\text{ bits}$ de `I2C_TXDATA` y verifica el ACK del esclavo. Si `RW_BIT == 1` (Lectura), recibe $8\text{ bits}$ desde el esclavo hacia `I2C_RXDATA` y responde con el valor configurado en `ACK_M`. Se autolimpia a `0`. |
| `[1]` | `START_CMD` | `R/W (AC)` | `1'b0` | **Comando de Inicio ($S$) y Direccionamiento:**<br>`1`: Genera la condición **START** ($\text{SDA}: 1 \rightarrow 0$ con $\text{SCL} = 1$) y transmite automáticamente los $8\text{ bits}$ del registro `I2C_ADDR` (`{SLAVE_ADDR[6:0], RW_BIT}`), capturando el ACK/NACK del esclavo en el $9.^\circ$ pulso. Se autolimpia a `0`. |
| `[0]` | `CORE_EN` | `R/W` | `1'b0` | **Habilitación Global del Periférico:**<br>`0`: Núcleo deshabilitado; contador de prescaler detenido y pines `SDA`/`SCL` en alta impedancia (`1'bZ`).<br>`1`: Núcleo habilitado y operativo. |

> **Nota de Operación Combinada (*Single-Shot*):** El hardware permite activar `START_CMD | DATA_CMD | STOP_CMD` simultáneamente (`0x0000_0017`) para ejecutar una transacción completa de $1\text{ byte}$ de forma autónoma, o bien activar los comandos paso a paso para transferencias en ráfaga (*Burst*).

---

### 4.2. `0x04` — `I2C_ADDR` (Registro de Dirección del Esclavo y Modo $R/\overline{W}$)

Contiene la trama de control de $8\text{ bits}$ que se carga en el registro de desplazamiento inmediatamente después de una condición START ($S$) o Repeated START ($Sr$).

* **Distribución de Bits:**
  * `[31:8]`: `RESERVED`
  * `[7:1]`: `SLAVE_ADDR[6:0]`
  * `[0]`: `RW_BIT`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:8]` | `RESERVED` | `RO` | `24'h000000` | **Reservado:** Lecturas retornan `0`. |
| `[7:1]` | `SLAVE_ADDR` | `R/W` | `7'b0000000` | **Dirección del Esclavo de $7\text{ bits}$ ($A_6 \dots A_0$):** Dirección física del dispositivo $I^2C$ destino (ej. `7'h68` para MPU6050, `7'h50` para EEPROM 24LC256). Se transmite empezando por el bit más significativo (`SLAVE_ADDR[6]`). |
| `[0]` | `RW_BIT` | `R/W` | `1'b0` | **Bit de Selección Lectura / Escritura ($R/\overline{W}$):**<br>`0` (**Write**): El maestro escribirá datos hacia el esclavo direccionado.<br>`1` (**Read**): El maestro leerá datos desde el esclavo direccionado. |

---

### 4.3. `0x08` — `I2C_TXDATA` (Registro de Datos de Transmisión)

Almacena el byte de datos que será serializado hacia el esclavo durante la fase de escritura.

* **Distribución de Bits:**
  * `[31:8]`: `RESERVED`
  * `[7:0]`: `TX_BYTE[7:0]`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:8]` | `RESERVED` | `RO` | `24'h000000` | **Reservado:** Lecturas retornan `0`. |
| `[7:0]` | `TX_BYTE` | `R/W` | `8'h00` | **Byte de Datos a Transmitir:** Dato de $8\text{ bits}$ que se copia al *Shift Register* cuando se ejecuta `DATA_CMD` en modo escritura (`RW_BIT == 0`). Se transmite en orden MSB primero (`TX_BYTE[7]` en el pulso $1$ de `SCL` hasta `TX_BYTE[0]` en el pulso $8$). Debe escribirse cuando `BUSY == 0`. |

---

### 4.4. `0x0C` — `I2C_RXDATA` (Registro de Datos de Recepción)

Contiene el último byte recibido desde el esclavo durante una fase de lectura.

* **Distribución de Bits:**
  * `[31:8]`: `RESERVED`
  * `[7:0]`: `RX_BYTE[7:0]`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:8]` | `RESERVED` | `RO` | `24'h000000` | **Reservado:** Lecturas retornan `0`. |
| `[7:0]` | `RX_BYTE` | `RO` | `8'h00` | **Byte de Datos Recibido:** Al finalizar los $8$ pulsos de lectura de `SCL` (`RW_BIT == 1`), el contenido del registro de desplazamiento se transfiere a este campo y se activa `DONE = 1` en `I2C_STATUS`. El bit `RX_BYTE[7]` corresponde al primer bit recibido (MSB) y `RX_BYTE[0]` al último (LSB). |

---

### 4.5. `0x10` — `I2C_STATUS` (Registro de Estado y Banderas del Bus)

Permite al procesador monitorear mediante *polling* o rutinas de interrupción el estado de la transferencia, errores de protocolo y el nivel eléctrico real de las líneas del bus.

* **Distribución de Bits:**
  * `[31:8]`: `RESERVED`
  * `[7]`: `SCL_IN`
  * `[6]`: `SDA_IN`
  * `[5]`: `STRETCH_DET`
  * `[4]`: `ARB_LOST`
  * `[3]`: `BUS_BUSY`
  * `[2]`: `ACK_ERR`
  * `[1]`: `DONE`
  * `[0]`: `BUSY`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:8]` | `RESERVED` | `RO` | `24'h000000` | **Reservado:** Lecturas retornan `0`. |
| `[7]` | `SCL_IN` | `RO` | `1'b1` | **Nivel Físico de `SCL`:** Refleja el estado lógico actual (ya sincronizado con doble Flip-Flop) de la línea `SCL` en el pin externo. En reposo debe leerse como `1`. |
| `[6]` | `SDA_IN` | `RO` | `1'b1` | **Nivel Físico de `SDA`:** Refleja el estado lógico actual sincronizado de la línea `SDA` en el pin externo. En reposo debe leerse como `1`. |
| `[5]` | `STRETCH_DET` | `RO` | `1'b0` | **Detección de *Clock Stretching*:**<br>`0`: Operación normal de reloj.<br>`1`: El maestro liberó `SCL` (`1'bZ`), pero el esclavo mantiene `SCL` forzado a `0` para pausar temporalmente la transferencia. |
| `[4]` | `ARB_LOST` | `W1C` | `1'b0` | **Pérdida de Arbitraje:**<br>`0`: Sin colisión detectada.<br>`1`: El maestro liberó `SDA` esperando un `1` durante el pulso alto de `SCL`, pero leyó `SDA_IN == 0`. La FSM aborta y libera el bus. Escribir `1` limpia la bandera. |
| `[3]` | `BUS_BUSY` | `RO` | `1'b0` | **Bus Ocupado (*Bus Busy*):**<br>`0`: Se ha emitido la condición STOP ($P$) y el bus se encuentra libre.<br>`1`: Se ha emitido una condición START ($S$) y aún no se ha cerrado la trama con un STOP ($P$). Permanece en `1` entre transferencias parciales para soportar *Repeated START* ($Sr$). |
| `[2]` | `ACK_ERR` | `W1C` | `1'b0` | **Error de Reconocimiento (*NACK Received*):**<br>`0`: El esclavo respondió con ACK (`0`) correctamente.<br>`1`: El esclavo respondió con **NACK (`1`)** en el $9.^\circ$ pulso de `SCL` tras enviar `I2C_ADDR` o `I2C_TXDATA`. Ante este error, el hardware genera automáticamente una condición STOP ($P$) para liberar el bus. Se limpia escribiendo `1` o al iniciar un nuevo `START_CMD`. |
| `[1]` | `DONE` | `W1C` | `1'b0` | **Operación Completada (*Transfer Done*):**<br>`0`: Ninguna operación recién finalizada o transferencia aún en curso.<br>`1`: El comando solicitado en `I2C_CTRL` finalizó. Se limpia escribiendo `1` en este bit o automáticamente cuando el procesador escribe un nuevo comando en `I2C_CTRL`. |
| `[0]` | `BUSY` | `RO` | `1'b0` | **Controlador Ocupado (*FSM Busy*):**<br>`0`: La FSM está en reposo (`IDLE`) o en estado de espera entre bytes (`HOLD`), lista para recibir un nuevo comando en `I2C_CTRL`.<br>`1`: La FSM está ejecutando activamente una condición $S$, $Sr$, $P$ o desplazando bits en el bus; no se deben modificar `I2C_ADDR`, `I2C_TXDATA` ni `I2C_PRESCALER`. |

---

### 4.6. `0x14` — `I2C_PRESCALER` (Registro Divisor de Frecuencia de Reloj)

Configura el divisor de reloj que genera las $4$ sub-fases internas ($Q_0, Q_1, Q_2, Q_3$) por cada período de la señal `SCL`.

* **Distribución de Bits:**
  * `[31:16]`: `RESERVED`
  * `[15:0]`: `PRESCALE_VAL[15:0]`

| Bits | Campo | Acceso | Reset | Descripción Detallada |
| :---: | :--- | :---: | :---: | :--- |
| `[31:16]` | `RESERVED` | `RO` | `16'h0000` | **Reservado:** Lecturas retornan `0`. |
| `[15:0]` | `PRESCALE_VAL` | `R/W` | `16'h003E` | **Valor del Divisor de Frecuencia ($16\text{ bits}$):** Define el número de ciclos del reloj del sistema ($f_{\text{CLK}}$) que dura cada uno de los $4$ cuadrantes de un período $T_{\text{SCL}}$. Su valor por defecto (`0x003E` $= 62_{10}$) configura $f_{\text{SCL}} \approx 100\text{ kHz}$ para un reloj de sistema de $25\text{ MHz}$. |

#### Ecuación de Cálculo del Prescaler

Dado que el generador de tiempos divide cada período de `SCL` en $4$ cuadrantes iguales para garantizar los tiempos de establecimiento ($t_{\text{SU;DAT}}$) y retención ($t_{\text{HD;DAT}}$), el valor a escribir en `I2C_PRESCALER` se calcula mediante:

$$\text{PRESCALE\_VAL} = \left\lfloor \frac{f_{\text{CLK}}}{4 \cdot f_{\text{SCL}}} \right\rfloor - 1$$

#### Tabla de Valores de Referencia para `I2C_PRESCALER`

| Frecuencia de Sistema ($f_{\text{CLK}}$) | Modo Estándar ($100\text{ kHz}$) | Modo Rápido ($400\text{ kHz}$) | Modo Rápido Plus ($1\text{ MHz}$) |
| :---: | :---: | :---: | :---: |
| $25\text{ MHz}$ | `62` (`0x003E`) | `15` (`0x000F`) | `5` (`0x0005`) |
| $50\text{ MHz}$ | `124` (`0x007C`) | `30` (`0x001E`) | `12` (`0x000C`) |
| $100\text{ MHz}$ | `249` (`0x00F9`) | `62` (`0x003E`) | `24` (`0x0018`) |

---

## 5. Integración RTL y Firmware (Referencia Rápida)

### 5.1. Decodificación de Direcciones en Verilog (`perip_i2c_master.v`)

Siguiendo la estructura de `perip_blink.v`, el decodificador de direcciones evalúa `addr[4:0]` cuando `cs == 1'b1`:

```verilog
localparam REG_CTRL      = 5'h00; // 0x00: I2C_CTRL
localparam REG_ADDR      = 5'h04; // 0x04: I2C_ADDR
localparam REG_TXDATA    = 5'h08; // 0x08: I2C_TXDATA
localparam REG_RXDATA    = 5'h0C; // 0x0C: I2C_RXDATA
localparam REG_STATUS    = 5'h10; // 0x10: I2C_STATUS
localparam REG_PRESCALER = 5'h14; // 0x14: I2C_PRESCALER
```

### 5.2. Definición de Registros y Máscaras de Bits en C (`i2c.h`)

```c
#ifndef I2C_MASTER_H
#define I2C_MASTER_H

#include <stdint.h>

typedef struct {
    volatile uint32_t CTRL;       // Offset 0x00: Control y Comandos
    volatile uint32_t ADDR;       // Offset 0x04: Dirección Esclavo [7:1] + R/W [0]
    volatile uint32_t TXDATA;     // Offset 0x08: Byte de Transmisión [7:0]
    volatile uint32_t RXDATA;     // Offset 0x0C: Byte de Recepción [7:0]
    volatile uint32_t STATUS;     // Offset 0x10: Estado y Banderas
    volatile uint32_t PRESCALER;  // Offset 0x14: Divisor de Frecuencia [15:0]
} I2C_TypeDef;

/* Máscaras de bits para I2C_CTRL (0x00) */
#define I2C_CTRL_CORE_EN        (1U << 0)
#define I2C_CTRL_START_CMD      (1U << 1)
#define I2C_CTRL_DATA_CMD       (1U << 2)
#define I2C_CTRL_REP_START_CMD  (1U << 3)
#define I2C_CTRL_STOP_CMD       (1U << 4)
#define I2C_CTRL_ACK_M          (1U << 5)
#define I2C_CTRL_IRQ_EN         (1U << 6)
#define I2C_CTRL_SWRST          (1U << 7)

/* Máscaras de bits para I2C_STATUS (0x10) */
#define I2C_STATUS_BUSY         (1U << 0)
#define I2C_STATUS_DONE         (1U << 1)
#define I2C_STATUS_ACK_ERR      (1U << 2)
#define I2C_STATUS_BUS_BUSY     (1U << 3)
#define I2C_STATUS_ARB_LOST     (1U << 4)
#define I2C_STATUS_STRETCH_DET  (1U << 5)
#define I2C_STATUS_SDA_IN       (1U << 6)
#define I2C_STATUS_SCL_IN       (1U << 7)

#endif /* I2C_MASTER_H */
```