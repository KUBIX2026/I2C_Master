# I2C Master Core para Consola de Videojuegos (Almacenamiento EEPROM)

## Introducción

Este repositorio contiene el desarrollo completo, estructurado y formal de un **controlador maestro I2C** implementado en Verilog. El proyecto está diseñado específicamente para integrarse dentro de una **consola de videojuegos**, cumpliendo la función crítica de gestionar la comunicación con una memoria **EEPROM** externa para garantizar el almacenamiento no volátil de los datos del sistema, puntajes altos (*high scores*), configuraciones y estados de juego.

---

## Criterios de Evaluación y Desarrollo del Proyecto

El proyecto está organizado metodológicamente bajo los siguientes 9 criterios de diseño digital y arquitectura de hardware:

1. **[C1 - Especificaciones y restricciones](C1%20Especificaciones%20y%20restricciones/)**
   * Definición de anchos de banda, velocidades del bus I2C (Standard/Fast mode), requerimientos de resistencias *pull-up* y restricciones temporales.
2. * *[C2 - Modelo y arquitectura del sistema](C2%20Modelo%20y%20arquitectura%20del%20sistema/)**
   * Diagramas de bloques y separación formal entre la Unidad de Control (FSMD) y el Camino de Datos (Datapath).
3. **C3 - Implementación técnica** *(Próximamente)*
   * Codificación RTL en Verilog modular, manejo seguro de líneas bidireccionales (*open-drain*) y sincronizadores de entrada (2FF).
4. **C4 - Integración hardware-software** *(Próximamente)*
   * Mapeo de E/S en memoria (MMIO) para la interacción del núcleo I2C con el microprocesador de la consola.
5. **C5 - Verificación, pruebas y depuración** *(Próximamente)*
   * Bancos de prueba (*testbenches*) autocontenidos y generación de archivos VCD para análisis con GTKWave.
6. **C6 - Integración y funcionamiento del sistema** *(Próximamente)*
   * Validación del generador de reloj maestro, gestión de ACK/NACK y manejo de errores de transmisión.
7. **C7 - Documentación y sustentación técnica** *(Próximamente)*
   * Reportes técnicos detallados, diagramas ASMD y especificaciones funcionales.
8. **C8 - Integración con memoria EEPROM** *(Próximamente)*
   * Pruebas de lectura y escritura reales orientadas a la persistencia de datos de los videojuegos.
9. **C9 - Despliegue y validación final** *(Próximamente)*
   * Síntesis física, análisis de tiempos estáticos y verificación en tarjeta FPGA.

---

## Estructura Actual del Repositorio

* `C1 Especificaciones y restricciones/` - Contiene las especificaciones técnicas, hojas de datos (*datasheets*) y la documentación preliminar de la EEPROM y el protocolo I2C.
* `C2 Modelo y arquitectura del sistema/` - Contiene los modelos arquitectónicos, la separación funcional de control y datapath, y los diagramas FSM.
