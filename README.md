
## Teoría de Operación: Protocolo I2C y Memoria EEPROM

A continuación se detallan los fundamentos teóricos del bus I2C y las características de la memoria externa, información clave para el desarrollo de la máquina de estados de nuestro periférico Master.

### Características Generales del Bus I2C

* **Tipo de Bus:** Bus de serie síncrono.
* **Topología:** Multi-master / Multi-slave.
* **Líneas físicas (2 hilos + Tierra):**

  * `SCL` (Serial Clock): Señal de reloj.
  * `SDA` (Serial Data): Señal de datos bidireccional.
  * `GND`: Tierra común.
* **Capacidad:** Permite direccionar hasta 127 dispositivos esclavos en el mismo bus.

### Roles en la Comunicación

* **Master (Maestro):**

  * Genera la señal de reloj (`SCL`).
  * Inicia y finaliza la comunicación.
  * Envía la dirección del esclavo con el que desea hablar.
* **Slave (Esclavo):**

  * Solo responde si es llamado por su dirección.
  * Nunca inicia la comunicación por sí mismo.

### Mecanismos de Control del Bus

* **Arbitraje:** Si dos Masters intentan enviar datos a la vez, gana el que envía un `0` lógico. Mientras tanto, el otro Master detecta que el estado de la línea `SDA` no coincide con lo que está transmitiendo y se retira automáticamente.
* **Clock Stretching (Estiramiento de reloj):** Un esclavo puede mantener la línea `SCL` en `0` (bajo) para pedirle más tiempo al Master antes de continuar.

<img width="465" height="185" alt="Figura 24-2" src="https://github.com/user-attachments/assets/4f20303b-a71c-4318-ace6-2ecc021f2cd1" />

**Figura 24-2. Diagrama de interconexión típica de un bus I2C entre un dispositivo Master y una memoria EEPROM.**

---

### Estructura de la Trama de Escritura/Lectura

La transmisión de datos sigue una secuencia estricta basada en el estado de las líneas, las cuales se mantienen en estado alto (High) mediante resistencias *pull-up* cuando están en reposo.

1. **Condición de START:**

   * La línea `SDA` baja a `0` mientras `SCL` sigue en alto (`1`).
   * Luego, el Master baja `SCL` para empezar a transmitir el primer byte.

2. **Transmisión de Dirección y R/W:**

   * El nivel de `SDA` solo puede cambiar cuando `SCL` está en bajo (`0`).
   * El receptor muestrea (lee) el bit cuando `SCL` está en alto (`1`). En este punto, `SDA` debe permanecer estable.

3. **Reloj y Datos (Bits):**

   * El reloj generado por el Master tiene **9 pulsos por byte**: 8 pulsos para los datos y 1 pulso extra para el bit de confirmación (ACK/NACK).

4. **Confirmación (ACK / NACK):**

   * Ocurre en el **9.º pulso de SCL**.
   * **ACK:** El Master suelta la línea `SDA` y el esclavo la lleva a `0` para confirmar la recepción.
   * **NACK:** Si la línea `SDA` se queda en `1` (alto) durante el 9.º pulso, significa que el esclavo no respondió o hubo un error.

5. **Condición de STOP:**

   * Es un flanco opuesto al START.
   * `SCL` sube primero a `1`. Luego, mientras `SCL` está en alto, `SDA` sube a `1`.

<img width="575" height="209" alt="Figura 24-3" src="https://github.com/user-attachments/assets/e01ef5dc-c76e-4641-a056-ec591213b34f" />

**Figura 24-3. Estados del protocolo I2C y condiciones de START, STOP, ACK/NACK y transferencia de datos.**

---

### Interfaz con Memoria EEPROM

El dispositivo esclavo objetivo de este proyecto es una memoria EEPROM (Memoria no volátil basada en celdas de transistores de compuerta flotante que se escribe y borra eléctricamente; a diferencia de la Flash, que borra por bloques/sectores).

* **Direccionamiento en el Bus I2C:**

  * Formato del byte de control: `1010` (Fijo para EEPROM) + `A2 A1 A0` (Pines de hardware) + `R/W` (Bit de Lectura/Escritura).
  * Los pines `A2-A0` permiten conectar hasta **8 memorias EEPROM** idénticas en el mismo bus.
* **Direccionamiento Interno:**

  * Se utiliza un *Word* para indicar la dirección interna de la celda donde se va a guardar/leer.
  * La EEPROM cuenta con un puntero interno que se autoincrementa tras leer o escribir cada byte.
* **Ciclo Interno de Escritura (Acknowledge Polling):**

  * Al recibir una orden de escritura, la EEPROM inicia un ciclo interno de guardado (típicamente tarda ~5 ms).
  * Durante este ciclo, la EEPROM **no responde** al bus.
  * El Master debe repetir constantemente las señales de `START` seguido de `Device Address + W` hasta que la EEPROM responda con un ACK, indicando que terminó de guardar.

---

## Comandos y Operaciones del I2C Master

Para implementar el protocolo I2C en la FPGA, la comunicación se organiza mediante diferentes comandos o estados de operación. Cada comando determina qué debe realizar el Master sobre las líneas `SCL` y `SDA`, así como la respuesta que debe esperar del dispositivo esclavo.

| Comando / Operación | Función                                       | Comportamiento de `SCL`      | Comportamiento de `SDA`                                        | Respuesta esperada                                  |
| ------------------- | --------------------------------------------- | ---------------------------- | -------------------------------------------------------------- | --------------------------------------------------- |
| `IDLE`              | Mantener el bus en reposo                     | `SCL = 1`                    | `SDA = 1` (línea liberada)                                     | Esperar una nueva operación                         |
| `START`             | Iniciar una comunicación                      | Permanece en `1`             | `1 → 0`                                                        | El esclavo detecta el inicio                        |
| `SEND_ADDRESS`      | Enviar dirección del esclavo y bit R/W        | Genera 8 pulsos              | El Master coloca los 8 bits de dirección                       | El esclavo responde con ACK/NACK                    |
| `CHECK_ACK`         | Comprobar confirmación                        | Genera el 9.º pulso          | El Master libera `SDA`; el esclavo puede llevarla a `0`        | `0 = ACK`, `1 = NACK`                               |
| `WRITE_DATA`        | Enviar un byte de datos                       | Genera 8 pulsos              | El Master coloca los bits del dato                             | El esclavo responde con ACK                         |
| `READ_DATA`         | Recibir un byte                               | Genera 8 pulsos              | El Master libera `SDA`; el esclavo coloca los bits             | El Master lee los bits                              |
| `SEND_ACK`          | Indicar que se desea continuar leyendo        | Genera el 9.º pulso          | Master coloca/libera según la interfaz para producir ACK (`0`) | El esclavo continúa                                 |
| `SEND_NACK`         | Indicar que se terminó la lectura             | Genera el 9.º pulso          | Master deja `SDA = 1` mediante liberación                      | El esclavo termina la transmisión                   |
| `RESTART`           | Reiniciar una comunicación sin liberar el bus | `SCL = 1`                    | `1 → 0`                                                        | Comienza una nueva fase                             |
| `STOP`              | Finalizar la comunicación                     | Primero `SCL → 1`            | `0 → 1` mientras `SCL = 1`                                     | Bus vuelve a IDLE                                   |
| `ACK_POLLING`       | Comprobar si la EEPROM terminó de escribir    | Genera los pulsos necesarios | Repite START + dirección + W                                   | ACK = EEPROM disponible; NACK = continuar esperando |
<img width="1441" height="1007" alt="imagen" src="https://github.com/user-attachments/assets/8e2b80b7-5599-4ca9-aa36-6fef49ea2f1f" />


**Figura 24-15. Secuencia de operación de un Master I2C durante la lectura de una memoria EEPROM.**

### Comportamiento de las señales durante un comando

El funcionamiento interno del Master puede entenderse como una secuencia de acciones sobre las dos líneas del bus. Por ejemplo, durante `SEND_ADDRESS`, la FPGA debe generar ocho pulsos de `SCL` y colocar un bit de la dirección sobre `SDA` en cada pulso. Después de transmitir los ocho bits, se ejecuta `CHECK_ACK`, donde el Master libera `SDA` y observa su estado durante el noveno pulso de `SCL`.

Una consideración importante para la implementación en FPGA es que `SDA` es una línea bidireccional. El Master no debe forzar directamente un nivel lógico alto; debe colocar la línea en estado de alta impedancia (`Z`) para liberarla y permitir que la resistencia *pull-up* establezca el nivel alto. Para transmitir un `0`, el dispositivo puede llevar la línea a nivel bajo.

Por lo tanto, conceptualmente:

`SDA = 0` → FPGA fuerza la línea a bajo.

`SDA = Z` → FPGA libera la línea y el *pull-up* permite que quede en alto.

Esto permite que tanto el Master como el Slave puedan participar en la comunicación sin que ambos intenten imponer simultáneamente niveles opuestos sobre el bus.

---

## Secuencia de escritura en la EEPROM

Una operación de escritura puede representarse mediante la siguiente secuencia:

**START → Device Address + W → ACK → Word Address → ACK → DATA → ACK → STOP**

En esta operación, el Master inicia la comunicación mediante `START`, envía la dirección de la EEPROM indicando una operación de escritura, espera el `ACK`, envía la dirección interna de memoria y posteriormente transmite el dato que será almacenado.

Después del último `ACK`, el Master genera `STOP`. La EEPROM comienza entonces su ciclo interno de escritura. Mientras este ciclo está activo, el dispositivo puede no responder a nuevas solicitudes. Por esta razón se utiliza el mecanismo de `ACK_POLLING`, mediante el cual el Master intenta nuevamente establecer comunicación hasta recibir un `ACK`.

**Secuencia de escritura:**

`START → CONTROL BYTE + W → ACK → WORD ADDRESS → ACK → DATA → ACK → STOP`

---

## Secuencia de lectura de la EEPROM

Para leer una posición específica de memoria se utiliza una operación de lectura aleatoria (*Random Read*). La secuencia general es:

**START → Device Address + W → ACK → Word Address → ACK → REPEATED START → Device Address + R → ACK → DATA → NACK → STOP**

Primero, el Master utiliza una operación de escritura para enviar la dirección interna de memoria que desea consultar. Una vez enviada esta dirección, genera un `REPEATED START` sin liberar el control del bus.

Después del `REPEATED START`, el Master vuelve a enviar la dirección del dispositivo, pero ahora con el bit `R/W = 1`, indicando que desea recibir información. La EEPROM coloca el dato sobre `SDA` mientras el Master genera los pulsos de `SCL`.

Al finalizar la recepción del último byte, el Master genera un `NACK` para indicar que no solicita más datos y finalmente genera la condición de `STOP`.

<img width="880" height="217" alt="Figura 24-4" src="https://github.com/user-attachments/assets/0801a9b0-a528-404d-9d3d-b046b4429fbb" />

**Figura 24-4. Mensaje típico I2C para la lectura de una memoria EEPROM en modo de dirección aleatoria.**

---

## Máquina de estados del I2C Master

Las operaciones anteriores pueden implementarse en la FPGA mediante una máquina de estados finitos (FSM). Cada estado representa una acción específica del protocolo y controla el comportamiento de `SCL`, `SDA` y los contadores internos.

Una posible organización de estados es:

`IDLE`

↓

`START`

↓

`SEND_ADDRESS`

↓

`CHECK_ACK`

↓

`SEND_MEMORY_ADDRESS`

↓

`CHECK_ACK`

↓

`WRITE_DATA` / `READ_DATA`

↓

`CHECK_ACK` / `SEND_ACK` / `SEND_NACK`

↓

`RESTART` (en una lectura)

↓

`STOP`

↓

`IDLE`

Para una escritura de EEPROM, después de `STOP` puede incorporarse el estado `ACK_POLLING`, encargado de comprobar cuándo la memoria termina su ciclo interno de escritura.

Esta organización permite que cada estado controle una parte específica de la trama I2C y facilita la implementación del protocolo mediante lógica secuencial en la FPGA.






# Esta plantilla no es de mi propiedad.

Toda la documentación y diseño de esta plantilla se encuentra dentro del siguiente repositorio el cual **no me pertenece** y es propiedad del usuario [@cicamargo](https://github.com/cicamargoba).

[Github de digital_UN](https://github.com/cicamargoba/digital_UN/tree/main/2026_1)
