## 1. Modelo y Arquitectura del Sistema (Criterio C2)

### 1.1 Diagrama de flujo de alto nivel. 
<img width="577" height="1202" alt="Diagrama flujo-I2C_Master_Flowchart drawio (3)" src="https://github.com/user-attachments/assets/b1c9b4f8-bcb7-4731-a5c5-e9f10ac89890" />

**Figura 1. Diagrama de flujo general protocolo I2C.**

### 1.2 Diagrama de bloques general.

<img width="736" height="477" alt="Captura desde 2026-10-05 05-35-00" src="https://github.com/user-attachments/assets/8f54781b-395f-4de5-840c-a57e91b409fa" />

Fuente de (a): Example of Single-Master Transmit/Receive Using I2C Bus Interface. https://www.renesas.com/en/document/dst/ipdatasheetd-6i2c?srsltid=AYCX9vLh13SW_b_1LoOmPjjA_okOy-O2Uos2ZTlyrLFNmu5KJe_EcAlm. abril 2010

* **Diagrama de bloques concreto (I²C Master propuesto)**



**Figura 2. Diagrama de bloques general protocolo I2C. (a) Diagrama general de referencia. (b) Diagrama concreto del I²C Master propuesto.**


* **MASTER:** Decide qué operación I²C realizar.



**Figura 2.1 Diagrama de bloques MASTER protocolo I2C.**

* **CONTROL UNIT:** Decide qué operación debe realizarse y en qué momento, mediante una máquina de estados (FSM).



**Figura 2.2 Diagrama de bloques CONTROL UNIT protocolo I2C.**

* **DATA PATH:** Contiene los registros, contadores, multiplexores y demás circuitos que manipulan, transmiten y reciben los datos durante la comunicación.


**Figura 2.3 Diagrama de bloques DATA PATH protocolo I2C.**

* **MASTER FINAL:** Controla y coordina la comunicación I²C con el dispositivo esclavo, generando las señales SCL y SDA necesarias para leer o escribir datos.
 
**Figura 2.4 Diagrama de bloques MASTER FINAL protocolo I2C.**

* **Generador de reloj:** Genera la señal de temporización `clk_i2c` y el pulso `tick` a partir del reloj del sistema y del valor de `prescaler`. Este valor define la frecuencia del bus (100 kHz / 400 kHz) y sincroniza al Control Unit, al Data Path y al Master Final.

**Figura 2.5 Diagrama de bloques del Generador de reloj.**

* **Registros de control y estado:** Almacenan la configuración de la operación y el estado del controlador. Permiten escribir y leer registros (`wr_en`, `rd_en`) y consultar el estado (`status[2:0]`), por lo que sirven de interfaz entre el sistema y el I²C Master.

  
**Figura 2.6 Diagrama de bloques de los Registros de control y estado.**

* **Secuencia de una transferencia I2C:** Muestra el orden de pasos de una transferencia, Corresponde solo al protocolo, no a un bloque del hardware.


  
**Figura 2.7 Secuencia de una transferencia I2C (solo protocolo).**


### 1.3 Diagrama de flujo operacional (Algoritmo de lectura y escritura del I²C Master).


**Figura 3. Diagrama de flujo del algoritmo de transferencia del I²C Master.**

### 1.4 Diagrama ASM.





**Figura 3.2 Diagrama de estados del I²C Master.**





