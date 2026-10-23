# Especificación funcional - Nodo de pH HELMO

Este documento describe el comportamiento funcional del firmware `firmware/nodo_ph/src/nodo_ph_casa_01.ino`, correspondiente al nodo dedicado de pH de la red HELMO. Su finalidad es complementar el código fuente con una explicación del principio de medición, la calibración empleada, la estructura del payload y la forma de integración del módulo dentro de la arquitectura distribuida del proyecto.

## Finalidad del módulo

El nodo de pH fue diseñado para medir una variable puntual de calidad del agua usando un sensor analógico conectado a un Heltec WiFi LoRa 32 V3. El firmware energiza temporalmente el sensor, toma múltiples lecturas ADC, filtra la señal, estima el voltaje del módulo y lo convierte a una medición de pH mediante una recta de calibración de dos puntos.

La elección de un nodo dedicado permite aislar la adquisición de pH respecto de otras variables, reduciendo acoplamiento entre sensores y facilitando tanto la validación del módulo como su despliegue en ubicaciones distintas de la red HELMO. Esta misma base puede reutilizarse para crear otros nodos de monitoreo en diferentes puntos geográficos o funcionales del sistema.

## Principio de medición

La lectura del sensor de pH se obtiene a partir de una salida analógica que es muestreada por el ADC del ESP32-S3. Posteriormente, el firmware aplica una calibración lineal de dos puntos basada en soluciones buffer de pH 4.00 y pH 7.00, siguiendo una práctica habitual para sensores analógicos de pH en entornos Arduino/ESP32.

El modelo implementado es de la forma `pH = m * V + b`, donde `m` y `b` se derivan de los voltajes medidos en los dos buffers de referencia. Este enfoque permite adaptar el nodo a calibraciones reales del montaje de laboratorio o de campo.

## Flujo de operación

1. Energizar el sensor de pH mediante MOSFET durante un tiempo de estabilización.
2. Tomar diez muestras ADC del canal analógico del sensor.
3. Ordenar las muestras de menor a mayor.
4. Descartar extremos y promediar las muestras centrales para reducir ruido.
5. Convertir el promedio ADC a voltaje.
6. Aplicar la recta de calibración para estimar el pH.
7. Limitar el valor al rango físico 0-14.
8. Empaquetar la información en una trama corta y transmitirla por LoRaWAN OTAA.

## Variables de salida

| Variable | Descripción |
|---|---|
| `voltajeSensor` | Voltaje calculado a partir del ADC del ESP32-S3. |
| `phVal` | Valor de pH estimado mediante calibración lineal de dos puntos. |
| `NODE_ID` | Identificador del nodo dentro de la red HELMO. |
| `NODO_ALIAS` | Alias descriptivo del punto de despliegue del nodo. |

## Integración con HELMO

El nodo de pH se integra a la arquitectura HELMO como un nodo de campo LoRaWAN OTAA que remite su payload a TTN/TTS y posteriormente al backend y a los servicios de visualización. Esta estructura es consistente con la separación por capas del proyecto, en la que cada nodo especializado produce telemetría compacta y trazable para ser interpretada por servicios superiores.

## Reutilización y escalabilidad

La base firmware del nodo de pH puede reutilizarse para crear nuevos nodos en otros sectores de la red HELMO, cambiando alias, credenciales OTAA, umbrales operativos o parámetros de calibración según el punto de instalación. Esto permite extender la cobertura del sistema sin alterar la lógica general de adquisición, empaquetado y transmisión ya definida por el proyecto.

## Valor documental para el registro

La presente especificación sirve como material auxiliar descriptivo del firmware y complementa el archivo fuente `.ino`. Su propósito es identificar el nodo de pH como una unidad software diferenciada y trazable dentro del sistema HELMO para soporte de expediente técnico y registro de derechos de autor.

## Autoría

La autoría del firmware del nodo de pH y de esta especificación técnica debe reconocerse a:

- Esteban Eduardo Escárraga Túquerres.
- Johan Felipe Ordoñez Jenoy.
- Dayerli Katerine Tamayo Solarte.
