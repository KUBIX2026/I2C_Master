## 1. Modelo y Arquitectura del Sistema (Criterio C2)

### 1.1 Diagrama de flujo de alto nivel. 
<img width="577" height="1202" alt="Diagrama flujo-I2C_Master_Flowchart drawio (3)" src="https://github.com/user-attachments/assets/b1c9b4f8-bcb7-4731-a5c5-e9f10ac89890" />

**Figura 1. Diagrama de flujo general protocolo I2C.**

### 1.2 Diagrama de bloques general.

<img width="736" height="477" alt="Captura desde 2026-10-05 05-35-00" src="https://github.com/user-attachments/assets/8f54781b-395f-4de5-840c-a57e91b409fa" />

Fuente de (a): Example of Single-Master Transmit/Receive Using I2C Bus Interface. https://www.renesas.com/en/document/dst/ipdatasheetd-6i2c?srsltid=AYCX9vLh13SW_b_1LoOmPjjA_okOy-O2Uos2ZTlyrLFNmu5KJe_EcAlm. abril 2010

* **Diagrama de bloques concreto (I²C Master propuesto)**

<img width="539" height="181" alt="image" src="https://github.com/user-attachments/assets/15919172-0095-4d7e-a9da-753ec793fbbb" />

**Figura 2. Diagrama de bloques general protocolo I2C. (a) Diagrama general de referencia. (b) Diagrama concreto del I²C Master propuesto.**


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

### 1.4 Diagrama ASM.
<img width="2750" height="5819" alt="carta_asm_master_i2c_v2" src="https://github.com/user-attachments/assets/01c09d56-618e-4e71-ade1-f8c04ba5d1b8" />

**Figura 3.1 Diagrama ASM del I²C Master.**

### 1.5 Diagrama de estados.
<img width="2760" height="5000" alt="diagrama_estados_control_i2c_v2" src="https://github.com/user-attachments/assets/5c6e4dfe-9f42-4756-9b1a-2594da3b5fb9" />

**Figura 3.2 Diagrama de estados del I²C Master.**





---

## Figura 1. Caja negra e interfaz del periférico I2C Master

<p align="center"><img width="2320" height="800" alt="image" src="https://github.com/user-attachments/assets/6f1748f4-a806-4ced-8baa-0976ad283057" />

</p>



**Descripción:** Esta figura muestra cómo se conecta el periférico I2C Master con el resto del sistema. La CPU se comunica con los registros CSR, que funcionan como intermediarios para enviar instrucciones y datos al núcleo `i2c_master_core`. Este núcleo controla la comunicación con la memoria EEPROM mediante las líneas SCL y SDA. También se observa el funcionamiento de los buffers open-drain, que permiten controlar las líneas del bus llevándolas a nivel bajo o liberándolas.

**Funcionamiento en orden:**

1. La CPU lee o escribe los registros CSR: CTRL, STATUS, DATA y PRESCALER, para configurar y controlar las operaciones del periférico.
2. Los registros CSR entregan al núcleo los comandos `cmd_start`, `cmd_write`, `cmd_read`, `cmd_stop`, además de la dirección del esclavo, el bit de lectura o escritura, los datos y la configuración del reloj.
3. El núcleo procesa las instrucciones y devuelve información mediante las señales `ready`, `bus_held`, `done`, `ack_err` y `rx_byte`.
4. Para controlar el bus, el núcleo utiliza `sda_low` y `scl_low`, que actúan sobre los buffers open-drain para llevar las líneas a nivel bajo o liberarlas.
5. Finalmente, las señales SCL y SDA llegan a la EEPROM. Las resistencias pull-up mantienen el nivel alto cuando las líneas se liberan, mientras que `sda_i` y `scl_i` permiten leer los niveles reales del bus.

---

## Figura 2. Arquitectura interna del i2c_master_core

<p align="center">
<img width="2080" height="1200" alt="image" src="https://github.com/user-attachments/assets/c268eb6e-38c8-4433-88ec-c244f841274e" />


</p>

**Descripción:** Esta figura muestra cómo está organizado internamente el núcleo I2C Master. Su funcionamiento se divide en dos partes: el controlador, encargado de decidir qué operación debe realizarse y cuándo, y el datapath, responsable de manipular los datos mediante registros, contadores, desplazadores y sincronizadores. Ambos trabajan coordinadamente para ejecutar las instrucciones recibidas.

**Funcionamiento en orden:**

1. El controlador recibe los comandos provenientes de la interfaz, junto con las señales de reloj `clk` y reinicio `rst`.
2. La máquina de estados FSM determina el siguiente estado y controla la fase de temporización `q`.
3. El controlador genera las señales de control: `load_addr`, `load_tx`, `clear_index`, `increment_index`, `scl_sel`, `sda_sel`, `shift_in`, `sample_ack`, `latch_rx` y `latch_nack`.
4. El datapath recibe estas órdenes y ejecuta las operaciones correspondientes, como cargar direcciones y datos, desplazar bits, actualizar contadores y controlar las salidas SDA y SCL.
5. El datapath devuelve información al controlador mediante señales como `q_tick`, `last_bit` y `ack_bit`.
6. Finalmente, el controlador genera `ready`, `bus_held`, `done` y `ack_err`, mientras que el datapath proporciona `rx_byte`, `sda_low` y `scl_low`.

---

## Figura 3. Máquina de estados finitos (FSM) del I2C Master

<p align="center">
<img width="1800" height="1040" alt="image" src="https://github.com/user-attachments/assets/86b62cbd-aeb5-4aca-8518-51661fdb3004" />

</p>

**Descripción:** Esta figura representa el recorrido que sigue el controlador durante una comunicación I2C. La máquina de estados está formada por ocho estados: IDLE, HOLD, START, SEND, GET_ACK, RECV, PUT_ACK y STOP. Cada estado cumple una función específica, desde esperar instrucciones hasta transmitir o recibir información y finalizar la comunicación.

**Funcionamiento en orden (escritura típica):**

1. El controlador comienza en IDLE, esperando una instrucción. Cuando recibe `cmd_start`, pasa al estado START.
2. En START se genera la condición de inicio del protocolo I2C. Al completar las fases de temporización, el controlador pasa a SEND.
3. En SEND se transmite el byte correspondiente, comenzando por el bit más significativo. El proceso se repite hasta completar los ocho bits y después se pasa a GET_ACK.
4. En GET_ACK se libera SDA para comprobar la respuesta del esclavo. Si recibe ACK, continúa hacia HOLD; si recibe NACK, activa el error y pasa automáticamente a STOP.
5. Desde HOLD, el controlador puede continuar según la instrucción recibida:
   - `cmd_write`: vuelve a SEND para transmitir otro byte.
   - `cmd_read`: pasa a RECV para recibir un byte y después a PUT_ACK para responder con ACK o NACK.
   - `cmd_start`: vuelve a START para generar un Repeated START.
   - `cmd_stop`: pasa a STOP para finalizar la comunicación.
6. Finalmente, STOP genera la condición de parada y el controlador regresa a IDLE, quedando preparado para una nueva operación.

---

## Figura 4. Diagrama ASMD del estado IDLE

<p align="center">
<img width="1280" height="914" alt="image" src="https://github.com/user-attachments/assets/159f4240-3e05-4d56-bbb9-352189697d45" />

</p>

**Descripción:** Esta figura muestra el estado inicial del controlador, en el que el I2C Master permanece esperando una nueva instrucción de inicio. Mientras se encuentra en IDLE, el periférico está disponible para recibir comandos, mantiene las líneas del bus liberadas y no ejecuta operaciones de transmisión o recepción. Solamente responde cuando recibe `cmd_start`.

**Funcionamiento en orden:**

1. El controlador entra en IDLE y establece `ready=1`, indicando que está disponible. También mantiene `bus_held=0` y las líneas del bus liberadas.
2. Comprueba si la señal `cmd_start` está activa.
3. Si `cmd_start=0`, no se inicia ninguna operación y el controlador permanece en IDLE.
4. Si `cmd_start=1`, ejecuta la acción A1: carga en el registro D la dirección del esclavo junto con el bit de lectura o escritura:

   $$D^{+} \leftarrow \{slave\_addr,\ rw\}$$

   También inicializa:

   $$i^{+}\leftarrow 0,\qquad q^{+}\leftarrow 0$$

   y libera SDA:

   $$sda\_low^{+}\leftarrow 0$$

5. Una vez preparados estos valores, el controlador pasa al estado START para comenzar la comunicación.

---

## Figura 5. Diagrama ASMD del estado HOLD

<p align="center"><img width="1720" height="1256" alt="image" src="https://github.com/user-attachments/assets/4c959394-63bf-45f0-9772-01d3b496f02f" />

</p>

**Descripción:** Esta figura representa el estado en el que el maestro mantiene la comunicación disponible para continuar enviando o recibiendo información. Durante HOLD, SCL permanece en nivel bajo y el controlador espera una nueva instrucción de la CPU. Dependiendo del comando recibido, puede iniciar otra transmisión, recibir datos, generar un nuevo START o finalizar la comunicación.

**Funcionamiento en orden:**

1. El controlador entra en HOLD y mantiene SCL en nivel bajo mientras espera una nueva instrucción.
2. Primero comprueba si `cmd_start=1`. Si es así, ejecuta A2: carga nuevamente la dirección del esclavo y el bit `rw`, libera SDA y pasa a START para generar un Repeated START.
3. Si no existe una instrucción START, comprueba `cmd_write`. Si está activa, ejecuta A3:

   $$D^{+}\leftarrow tx\_byte$$

   Después pasa a SEND para transmitir el byte.
4. Si tampoco existe una instrucción WRITE, comprueba `cmd_read`. Si está activa, ejecuta A4:

   $$nl^{+}\leftarrow nack\_last$$

   Libera SDA y pasa a RECV para comenzar a recibir información.
5. Si no hay una instrucción de lectura, comprueba `cmd_stop`. Cuando está activa, ejecuta A5 y pasa a STOP.
6. Si ninguna instrucción está activa, permanece en HOLD esperando el siguiente comando.

---

## Figura 6. Diagrama ASMD del estado START

<p align="center">
<img width="1040" height="1412" alt="image" src="https://github.com/user-attachments/assets/33eaccf0-18fc-4638-81ea-4c8d63128dc1" />

</p>

**Descripción:** Esta figura explica cómo el maestro genera la condición START del protocolo I2C. Para hacerlo, el controlador sigue cinco fases de temporización, identificadas como S0, S1, S2, S3 y S4. Durante estas fases modifica las señales SDA y SCL hasta producir la condición de inicio, que ocurre cuando SDA pasa de nivel alto a nivel bajo mientras SCL permanece en nivel alto. Cada avance depende de que `q_tick=1`.

**Funcionamiento en orden:**

1. S0 (`q=0`): se libera SCL para llevarla a nivel alto.
2. S1 (`q=1`): se mantiene la señal mientras se espera completar el intervalo correspondiente.
3. S2 (`q=2`): se lleva SDA a nivel bajo mientras SCL permanece en nivel alto. Esto genera la condición START.
4. S3 (`q=3`): se mantiene la condición generada durante el intervalo correspondiente.
5. S4 (`q=4`): se lleva SCL a nivel bajo y se completa la secuencia de inicio.
6. Una vez terminada esta operación, el controlador pasa a SEND para comenzar a transmitir la dirección o los datos correspondientes.

---

## Figura 7. Diagrama ASMD del estado SEND

<p align="center">
<img width="800" alt="figura_07_asmd_send" src="https://github.com/user-attachments/assets/728391dc-00d9-4424-a81d-26664af26f3a" />
</p>

**Descripción:** Esta figura muestra cómo el maestro transmite un byte de información hacia el dispositivo esclavo. La transmisión se realiza bit por bit a través de la línea SDA, comenzando por el bit más significativo (MSB). El controlador utiliza las fases de temporización para preparar cada bit, permitir que el esclavo lo lea y avanzar hasta completar los ocho bits del byte.

**Funcionamiento en orden:**

1. El controlador entra en SEND con el byte que debe transmitir almacenado en el registro D.
2. W0: coloca sobre SDA el bit correspondiente del registro, siguiendo el orden:

   $$D[7-i]$$

   SCL permanece en nivel bajo para preparar el dato.
3. W1: libera SCL para llevarla a nivel alto y permitir que el esclavo observe el valor presente en SDA.
4. W2: mantiene SCL en nivel alto durante el intervalo correspondiente, permitiendo que el dato permanezca estable.
5. W3: comprueba si todavía quedan bits por transmitir. Si $i<7$, incrementa el contador:

   $$i^{+}\leftarrow i+1$$

   Lleva SCL a nivel bajo y repite el proceso.
6. W4: cuando $i=7$, significa que ya se transmitieron los ocho bits. El controlador lleva SCL a nivel bajo y pasa a GET_ACK para comprobar la respuesta del esclavo.

---

## Figura 8. Diagrama ASMD del estado GET_ACK

<p align="center">
<img width="800" alt="figura_08_asmd_get_ack" src="https://github.com/user-attachments/assets/799317a6-556d-4c2b-b76e-a4aebbfd1aec" />
</p>

**Descripción:** Esta figura representa el momento en el que el maestro espera la confirmación del dispositivo esclavo después de transmitir un byte. Para permitir que el esclavo responda, el maestro libera SDA y posteriormente lee su valor cuando SCL se encuentra en nivel alto. Dependiendo de la respuesta recibida, la comunicación puede continuar o finalizar mediante una condición STOP.

**Funcionamiento en orden:**

1. El controlador entra en GET_ACK después de completar la transmisión de los ocho bits.
2. G0: libera SDA para que el dispositivo esclavo pueda colocar su respuesta en el bus.
3. G1: libera SCL para llevarla a nivel alto y permitir que el esclavo presente su respuesta.
4. G2: el maestro lee el valor de SDA y lo almacena:

   $$ack\_bit^{+}\leftarrow sda\_i$$

5. Si el valor recibido es `ack_bit=0`, significa que el esclavo respondió ACK. En G3, el controlador lleva SCL a nivel bajo, activa `done=1` y regresa a HOLD.
6. Si el valor recibido es `ack_bit=1`, significa que el esclavo respondió NACK. En G4, el controlador lleva SCL a nivel bajo, activa `ack_err=1` y pasa a STOP para finalizar la comunicación.

---

## Figura 9. Diagrama ASMD del estado RECV

<p align="center">
<img width="800" alt="figura_09_asmd_recv" src="https://github.com/user-attachments/assets/317c7ea2-afe3-4cba-ba2c-a964d64eb0c6" />
</p>

**Descripción:** Esta figura muestra cómo el maestro recibe un byte enviado por el dispositivo esclavo. En este proceso, el maestro libera SDA para que el esclavo pueda colocar los bits en el bus. El controlador lee cada bit y lo almacena progresivamente en el registro de desplazamiento `rx_sh`, hasta completar los ocho bits correspondientes al dato recibido.

**Funcionamiento en orden:**

1. El controlador entra en RECV y prepara el proceso de recepción del byte.
2. V0: espera el intervalo de temporización correspondiente, manteniendo SDA liberada para que el esclavo pueda enviar información.
3. V1: libera SCL para llevarla a nivel alto y permitir que el dato presente en SDA pueda ser leído.
4. V2: el maestro toma el valor de SDA y lo introduce en el registro de desplazamiento:

   $$rx\_sh^{+}\leftarrow\{rx\_sh[6:0],\ sda\_i\}$$

   De esta manera, cada nuevo bit se incorpora al registro.
5. V3: comprueba si todavía faltan bits por recibir. Si $i<7$, incrementa el contador, lleva SCL a nivel bajo y repite el proceso.
6. V4: cuando $i=7$, significa que ya se recibieron los ocho bits del byte. El controlador lleva SCL a nivel bajo y pasa a PUT_ACK para responder al esclavo.

---

## Figura 10. Diagrama ASMD del estado PUT_ACK

<p align="center">
<img width="800" alt="figura_10_asmd_put_ack" src="https://github.com/user-attachments/assets/c0b84bf9-27ff-468f-842e-26cb7df47d39" />
</p>

**Descripción:** Esta figura muestra cómo el maestro responde después de recibir un byte del dispositivo esclavo. Una vez completada la recepción, el maestro debe indicar si desea continuar recibiendo información o si ese era el último byte solicitado. Para ello, utiliza la señal `nack_last`, que determina si debe enviar una respuesta ACK o NACK mediante la línea SDA.

**Funcionamiento en orden:**

1. El controlador entra en PUT_ACK después de recibir los ocho bits del dato.
2. K0: prepara la respuesta mediante:

   $$sda\_low^{+}\leftarrow\neg nl$$

   Si $nl=0$, el maestro lleva SDA a nivel bajo y envía ACK. Si $nl=1$, libera SDA y envía NACK.
3. K1: libera SCL para llevarla a nivel alto y permitir que el esclavo observe la respuesta del maestro.
4. K2: mantiene las señales durante el intervalo de temporización correspondiente.
5. K3: lleva SCL a nivel bajo, transfiere el dato recibido al registro de salida:

   $$rx\_byte^{+}\leftarrow rx\_sh$$

   y activa `done=1` para indicar que terminó la recepción del byte.
6. Finalmente, el controlador regresa a HOLD y queda esperando una nueva instrucción.

---

## Figura 11. Diagrama ASMD del estado STOP

<p align="center">
<img width="800" alt="figura_11_asmd_stop" src="https://github.com/user-attachments/assets/a92688d5-738e-4483-8ba1-814763384ccf" />
</p>

**Descripción:** Esta figura representa el proceso mediante el cual el maestro finaliza la comunicación con el dispositivo esclavo. Para generar la condición STOP, el controlador realiza cinco fases de temporización. Durante ellas organiza los niveles de SDA y SCL hasta producir el cambio de SDA de nivel bajo a nivel alto mientras SCL permanece en nivel alto, indicando que la comunicación ha terminado.

**Funcionamiento en orden:**

1. El controlador entra en STOP y comienza la secuencia de finalización.
2. X0: lleva SDA a nivel bajo para preparar la condición de parada.
3. X1: libera SCL para llevarla a nivel alto.
4. X2: mantiene SDA en nivel bajo y SCL en nivel alto durante el intervalo correspondiente.
5. X3: libera SDA mientras SCL permanece en nivel alto. Este cambio genera la condición STOP del protocolo I2C.
6. X4: activa `done=1` para indicar que la operación ha finalizado y regresa a IDLE, donde queda preparado para recibir nuevas instrucciones.

---

## Figura 12. Diagrama temporal de escritura de 0xA6 en la palabra 0x10 de la EEPROM

<p align="center">
<img width="2296" height="840" alt="image" src="https://github.com/user-attachments/assets/3711d7b4-00d1-43a1-a5d9-ad84fb33bb00" />

</p>

**Descripción:** Esta figura muestra un ejemplo completo de cómo el I2C Master escribe el dato `0xA6` en la dirección interna `0x10` de la memoria EEPROM, cuyo identificador de esclavo es `0x50`. A través de las señales SCL y SDA se observa cómo se desarrolla la comunicación, desde la condición START hasta la condición STOP, incluyendo el envío de la dirección del dispositivo, la dirección de memoria, el dato y las respuestas ACK. Es una representación lógica de la operación y no una captura de simulación.

**Funcionamiento en orden:**

1. **Inicio de la comunicación:** La CPU activa `cmd_start` y el maestro genera la condición START. Después transmite la dirección del esclavo `0x50` junto con el bit de escritura W. Esto forma el byte `0xA0`, que se envía por SDA. El esclavo responde con ACK para confirmar que recibió su dirección. Esta primera parte ocupa 41 períodos Q: 5 Q para START, 32 Q para la dirección y 4 Q para ACK.
2. **Espera entre instrucciones:** Una vez recibida la confirmación, el controlador pasa a HOLD, donde mantiene SCL en nivel bajo y espera la siguiente instrucción.
3. **Envío de la dirección de memoria:** La CPU activa `cmd_write` y proporciona el valor `0x10`, que indica la posición interna de la EEPROM donde se almacenará el dato. El maestro transmite este byte y espera la respuesta ACK del esclavo. Esta operación ocupa 36 períodos Q.
4. **Nueva espera:** Después de recibir ACK, el controlador regresa a HOLD para esperar otro comando.
5. **Envío del dato:** La CPU activa nuevamente `cmd_write`, esta vez con el valor `0xA6`. El maestro transmite el byte a la EEPROM y espera su respuesta ACK. Esta operación ocupa otros 36 períodos Q.
6. **Preparación de la finalización:** El controlador vuelve a HOLD después de recibir la confirmación del esclavo.
7. **Finalización de la comunicación:** La CPU activa `cmd_stop` y el maestro genera la condición STOP, liberando SDA mientras SCL permanece en nivel alto. Esta operación ocupa 5 períodos Q y marca el final de la escritura.

**Tiempo total de la operación:**

- START: 5 Q.
- Dirección del esclavo y ACK: 36 Q.
- Dirección interna de memoria y ACK: 36 Q.
- Dato y ACK: 36 Q.
- STOP: 5 Q.

Por lo tanto, el tiempo total es:

$$Q_{total}=5+36+36+36+5=118\ Q$$

Considerando 63 ciclos de reloj por período Q:

$$N_{ciclos}=118\times63=7434$$

Para un reloj de entrada de 25 MHz:

$$t=\frac{7434}{25\times10^6}=\mathbf{297.36\ \mu s}$$

Este tiempo no incluye los intervalos de permanencia en HOLD.




Diseño del controlador I²C Master

Descripción general

En esta sección se presenta el diseño arquitectónico del controlador I²C Master desarrollado para el proyecto KUBIX. Se documenta la comunicación entre la CPU y el periférico, la organización interna del controlador y la distribución de las funciones de control y procesamiento de datos.

El diseño se divide en tres diagramas que permiten comprender progresivamente la arquitectura propuesta.

1. Arquitectura CPU – I²C Master

"Arquitectura CPU-I2C" 

<img width="1547" height="976" alt="image" src="https://github.com/user-attachments/assets/7e7dd90e-597d-41d1-b58f-e6bfb2c404e0" />


Este diagrama representa la conexión entre la CPU y el periférico I²C Master. Se muestra el uso de un decodificador de direcciones (Address Decoder) para identificar los accesos destinados al controlador, así como las señales de control y datos que permiten configurar e iniciar las operaciones.

También se presenta la interfaz de registros CSR (Control and Status Registers), mediante la cual la CPU proporciona parámetros de transmisión y recibe información sobre el estado de las operaciones.

2. Arquitectura interna: Controller y Datapath

"Arquitectura Controller y Datapath" 

<img width="1500" height="941" alt="image" src="https://github.com/user-attachments/assets/7c44a955-b2a5-47cc-a97a-3dcea24863e0" />


El controlador I²C Master se organiza en dos bloques principales:

- Controller (FSM): encargado de gestionar la secuencia de operaciones mediante una máquina de estados finitos. Interpreta los comandos recibidos y genera las señales de control necesarias para coordinar el funcionamiento del sistema.
- Datapath: encargado del almacenamiento, desplazamiento y procesamiento de los datos, además de la generación de las señales asociadas a la comunicación I²C.

Ambos bloques trabajan de manera coordinada. El Controller envía señales de control al Datapath y recibe de este información de estado para determinar las transiciones entre estados.

3. Arquitectura interna del Datapath

"Detalle del Datapath" 
<img width="1509" height="1122" alt="image" src="https://github.com/user-attachments/assets/461b9966-1d8a-47c5-8bb0-547dc2dfa544" />


Este diagrama presenta con mayor detalle la organización interna del Datapath y el flujo de información entre sus componentes.

Se identifican los siguientes bloques funcionales:

- Registro D: almacena temporalmente la dirección del dispositivo esclavo y el byte que se desea transmitir.
- Contador i: lleva el seguimiento de los bits procesados durante las operaciones de transmisión y recepción.
- Timer: genera las señales temporales necesarias para coordinar las operaciones del protocolo, a partir del prescaler configurado.
- Registro rx_sh: permite recibir y almacenar secuencialmente los bits provenientes de la línea SDA.
- Selector SDA/SCL: determina el comportamiento de las señales de control asociadas a las líneas de comunicación.
- Registros de salida: almacenan los datos recibidos y las señales de resultado que serán utilizadas por el controlador y la interfaz externa.

Estado del diseño

Los diagramas representan la arquitectura funcional propuesta para el controlador I²C Master de KUBIX. Constituyen la base para la siguiente etapa de desarrollo, correspondiente a la revisión de señales, validación de la lógica de control y posterior descripción del hardware mediante código HDL.




## 2. Almacenamiento, Guardado y Recuperación de Puntuaciones (*Scores*)

### 2.1. ¿Dónde se almacena el Score?
El puntaje del juego posee dos niveles de almacenamiento:
1. **Volátil (RAM del Procesador):** Durante el transcurso de la partida, la puntuación se actualiza en tiempo real en registros y variables alojadas en la memoria RAM interna del procesador SoC.
2. **No Volátil (Memoria EEPROM):** Al finalizar la partida, el puntaje es transferido mediante el periférico `i2c_master` hacia una memoria EEPROM externa, garantizando que los récords persistan aunque el sistema se apague.
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
| **Hardware** (`i2c_master.v` + `i2c_master_core.v`) | Generación de reloj `SCL` y cuadrantes $Q_0\text{–}Q_3$, serialización/deserialización MSB-first de 8 bits, conteo del pulso 9 (`ACK/NACK`), sincronización y control *Open-Drain*. | Requiere precisión determinista a nivel de microsegundos sin bloquear el ciclo de instrucciones de la CPU. |
| **Software** (`i2c.c`, `i2c.h` y Aplicación) | Configuración del `PRESCALER`, segmentación del *Score* en bytes, direccionamiento de celdas de la EEPROM, control de tiempos $t_{WR}$ y lógica de renderizado en pantalla. | Otorga flexibilidad para modificar estructuras de datos o protocolos de visualización sin resintetizar la FPGA. |

### 3.2. Análisis de Alternativas Arquitectónicas
* **Periférico Hardware RTL (Elegida) vs. *Bit-Banging* por Software:** Descarga a la CPU de retardos bloqueantes de $\sim 90\text{ }\mu\text{s}$ por byte, garantizando márgenes temporales estrictos para el bus $I^2C$.
* **Diseño Síncrono con Habilitación de 4 Fases `tick_4x` (Elegida) vs. Relojes Múltiples:** Opera 100% bajo un único dominio de reloj (`clk`), evitando problemas de cruce de dominios (*CDC*) y garantizando estabilidad en el muestreo.
* **Protocolo $I^2C$ de 2 Hilos (Elegida) vs. SPI:** Optimiza el uso de pines en la FPGA a solo 2 líneas (`SDA`, `SCL`) y provee confirmación nativa de recepción (`ACK/NACK`) igual de ideal para memorias de almacenamiento de puntajes.

---
