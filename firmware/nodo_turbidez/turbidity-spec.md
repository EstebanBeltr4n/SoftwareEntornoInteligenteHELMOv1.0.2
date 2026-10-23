# Especificación funcional - Nodo de turbidez PTAP

Este documento describe el comportamiento funcional del firmware `firmware/nodo_turbidez/src/nodo_turbidez_ptap.ino`, correspondiente al nodo de turbidez del sistema HELMO. Su finalidad es complementar el código fuente con una descripción técnica del principio de adquisición, la lógica de estimación de NTU, los estados definidos y la integración del módulo con la arquitectura LoRaWAN del proyecto.

## Finalidad del módulo

El nodo de turbidez se diseñó para medir una variable de calidad de agua mediante la lectura analógica de un sensor óptico o equivalente de turbidez. El firmware convierte la señal capturada por el ADC del ESP32-S3 a una estimación de voltaje y luego a un valor aproximado de NTU, generando información compacta para ser transmitida dentro de la red HELMO.

## Principio de medición

En sensores de turbidez analógicos para ESP32, una práctica común consiste en convertir la lectura ADC a voltaje y luego aplicar una ecuación de calibración, lineal o no lineal, derivada de pruebas del propio sistema. Diversas referencias técnicas muestran justamente este enfoque y recomiendan calibrar con puntos reales del montaje, agua clara y condiciones de mayor turbidez para mejorar la validez del modelo.

La versión documental de este nodo usa un modelo lineal configurable definido por `NTU_A` y `NTU_B`, junto con un `DIVISOR_FACTOR` para compensar la etapa de acondicionamiento analógico cuando existe divisor resistivo o adaptación de señal entre el sensor y el ADC del microcontrolador.

## Flujo de operación

1. Energizar la etapa de sensado mediante MOSFET.
2. Tomar múltiples lecturas ADC del sensor de turbidez.
3. Promediar las muestras para reducir ruido.
4. Convertir el valor ADC a voltaje en el pin del ESP32-S3.
5. Reconstruir el voltaje del sensor usando el factor de corrección del divisor.
6. Estimar la turbidez en NTU con la recta de calibración configurada.
7. Clasificar el estado de turbidez en categorías discretas.
8. Empaquetar la información en una trama de 6 bytes para envío LoRaWAN.

## Estados definidos

El firmware define cuatro estados lógicos de turbidez:

- `0`: muy_baja.
- `1`: baja.
- `2`: media.
- `3`: alta.

Estos estados son una abstracción operativa que facilita la representación compacta del punto monitoreado y su posterior interpretación en dashboards, alarmas o servicios backend del sistema HELMO.

## Variables de salida

| Variable | Descripción |
|---|---|
| `adcRaw` | Lectura bruta del ADC del ESP32-S3.
| `adcVoltage` | Voltaje equivalente medido en el pin ADC. |
| `sensorVoltage` | Voltaje reconstruido del sensor considerando divisor o acondicionamiento. |
| `turbidityNtu` | Estimación de turbidez en unidades NTU. |
| `turbidityState` | Clasificación categórica de la turbidez. |

## Integración con HELMO

El nodo se integra a la arquitectura HELMO como un componente de adquisición distribuida especializado en una variable de calidad del agua. Su salida puede ser consumida por TTN/TTS, backend, base de datos y servicios de visualización, manteniendo la misma lógica de separación por capas que ya existe en los otros nodos del proyecto.

## Valor documental para el registro

La presente especificación sirve como material auxiliar descriptivo del firmware y complementa el archivo fuente `.ino`. Su propósito es identificar el nodo de turbidez como una unidad software diferenciada dentro del sistema HELMO para soporte del expediente técnico y del registro de derechos de autor.

## Autoría

La autoría del firmware del nodo de turbidez y de esta especificación técnica debe reconocerse a:

- Esteban Eduardo Escárraga Túquerres.
- Johan Felipe Ordoñez Jenoy.
- Dayerli Katerine Tamayo Solarte.
