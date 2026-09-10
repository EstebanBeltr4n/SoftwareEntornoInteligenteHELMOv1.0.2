# HELMO - Instalación del entorno de trabajo

Este documento describe el entorno de trabajo recomendado para compilar firmware, organizar el repositorio y operar la capa software asociada a HELMO. Su finalidad es dejar trazabilidad sobre las herramientas mínimas necesarias para desarrollar, mantener y desplegar los componentes del sistema dentro de un flujo profesional y reproducible.

## Herramientas principales

Para el trabajo técnico de HELMO se recomienda disponer al menos de los siguientes elementos:

- Visual Studio Code como IDE principal del repositorio y terminal integrada para Git.
- Arduino IDE como entorno de compilación para nodos Heltec WiFi LoRa 32 V3, dado que Heltec recomienda Arduino y proporciona soporte y ejemplos LoRaWAN para esa placa.
- Git y GitHub para control de versiones, trazabilidad y publicación del repositorio técnico.
- Node.js y npm para componentes backend o servicios asociados al sistema HELMO.
- MongoDB/Atlas para persistencia de datos históricos y consultas de telemetría, según la arquitectura conceptual adjunta.
- Grafana y mecanismos de exposición o acceso web como parte de la visualización del sistema, según la arquitectura del proyecto.

## Estructura base del repositorio

El repositorio de HELMO se ha venido organizando en bloques separados de `firmware/`, `hardware/` y `docs/`, con documentación individual por nodo y documentación global del sistema. Esta separación es adecuada para proyectos embebidos y facilita dejar evidencia organizada para fines académicos, técnicos y de derechos de autor.

Una estructura general recomendada es:

```text
helmo-software/
├── firmware/
├── hardware/
├── docs/
├── README.md
├── CHANGELOG.md
└── .gitignore
```

## Instalación para firmware

Para la capa firmware, se debe instalar Arduino IDE con soporte para la placa Heltec WiFi LoRa 32 V3 y las librerías requeridas por la base LoRaWAN usada por los nodos. En el código de HELMO ya se utilizan cabeceras como `LoRaWan_APP.h` y `HT_SSD1306Wire.h`, coherentes con el ecosistema de desarrollo documentado por Heltec para esta familia de placas.

Cada nodo del sistema debe mantenerse en su carpeta específica dentro de `firmware/`, por ejemplo `nodo_ph`, `nodo_turbidez` o `nodo_nivel`, junto con su archivo `.ino`, README y especificaciones de payload y funcionamiento. Esta separación modular simplifica compilación, pruebas y trazabilidad del software de campo.

## Instalación para backend y servicios

La arquitectura conceptual de HELMO indica que la infraestructura superior del sistema contempla servidor IoT/gateway, base de datos, TTN/TTS y servicio web. En consecuencia, el entorno de trabajo debe prever acceso o despliegue de componentes backend, almacenamiento y herramientas de visualización o paneles web.

En el contexto del proyecto también se ha mencionado experiencia con Node.js, MongoDB, React, backend API, pm2 y Grafana, lo que refuerza la conveniencia de mantener el entorno de desarrollo orientado a una arquitectura SaaS moderna y separada por servicios. 

## Recomendaciones operativas

- Mantener credenciales sensibles fuera de documentos destinados a expediente o repositorio público.
- Usar archivos documentales por módulo para conservar consistencia entre firmware y soporte técnico.
- Registrar cambios importantes en `CHANGELOG.md` o documentos de versión del sistema.
- Conservar capturas, diagramas y descripciones en `hardware/` y `docs/` como apoyo documental del proyecto.

## Valor para derechos de autor

Este documento no reemplaza el código fuente, pero complementa la identificación del entorno técnico en el que HELMO fue concebido, organizado y mantenido. Como material auxiliar, ayuda a mostrar que el sistema posee estructura de desarrollo, herramientas definidas y una metodología de implementación técnica consistente.

## Autoría

La autoría de la documentación de instalación y entorno de trabajo de HELMO debe reconocerse a:

- Esteban Eduardo Escárraga Túquerres.
- Johan Felipe Ordoñez Jenoy.
- Dayerli Katerine Tamayo Solarte.
