# Changelog

Todos los cambios relevantes de este proyecto serán documentados en este archivo. El formato adopta la convención de Keep a Changelog y se utiliza como registro legible por humanos del avance técnico y documental del sistema HELMO.

## [Unreleased]

Sin cambios pendientes de publicación.

## [1.0.0] - 2026-09-22

### Added
- Firmware completo de los cuatro nodos de campo: nodo concentrador multisensor (nivel, turbidez, pH) y nodos dedicados de pH, turbidez y nivel, cada uno con código fuente `.ino`, README, especificación funcional y especificación de payload.
- Backend de ingestión: API HTTP con autenticación básica sobre MongoDB (`backend/api`) y puente MQTT-MongoDB con persistencia dual local/Atlas (`backend/helmo-bridge`).
- Decodificador de payload uplink para The Things Stack (`integrations/ttn/uplink_decoder.js`), con soporte para nodo monolítico, nodo dedicado de pH y formato heredado.
- Documentación de despliegue del MVP: exposición de Grafana mediante ngrok, y evidencias fotográficas del dashboard, del hardware en campo y de pruebas de conectividad LoRaWAN.
- Documentación legal complementaria en `legal/`: autoría, titularidad, alcance documental y versión declarada.
- Anexo técnico de soporte para registro de derechos de autor ante la DNDA en `legal/registro_dnda/`.

### Changed
- Se consolidó la organización del repositorio para presentar HELMO como obra técnica integral con componentes de firmware, backend, integración TTN, documentación y soporte legal.
- Se corrigieron los archivos `.md` del repositorio, eliminando marcadores de referencia internos y normalizando formato.

### Security
- Las credenciales y datos sensibles utilizados en firmware documental fueron neutralizados o reemplazados por valores nulos en los archivos preparados para soporte de derechos de autor.
- Se excluyeron del historial capturas con información sensible (contraseña de red) detectadas durante la revisión de evidencias fotográficas.

## [0.1.0] - 2026-09-04

### Added
- Versión inicial del repositorio documental de HELMO preparada para estructuración técnica, organización de evidencia y soporte de expediente de derechos de autor.
- Archivo `README.md` raíz y archivo `LICENSE` como base descriptiva y jurídica del proyecto.
- Carpeta `docs/` con documentos generales del sistema y referencia a la arquitectura conceptual adjunta.
- Carpeta `legal/` con documentos complementarios al archivo de licencia.

### Notes
- Esta versión corresponde a una línea base documental y técnica del proyecto HELMO, útil para continuidad del desarrollo y control de cambios futuros.
