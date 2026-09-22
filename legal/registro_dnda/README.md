# Material para registro ante la DNDA

Esta carpeta reúne los archivos listos para el trámite de registro de software ante la Dirección Nacional de Derecho de Autor (DNDA), siguiendo el criterio de que basta con adjuntar **uno** de los tres elementos habituales (descripción, extracto de código, manual) — aquí se incluyen los tres combinados en un solo anexo, más el texto corto exigido por el formulario.

## Archivos

- **`descripcion_corta.md`** — texto de ≤500 caracteres para el campo de descripción corta del formulario. Longitud verificada.
- **`memoria_tecnica_dnda.md`** — fuente en Markdown del anexo técnico completo: identificación de la obra, descripción funcional detallada, extracto representativo de código (firmware, decodificador TTN y backend) y manual de usuario/instalación.
- **`memoria_tecnica_dnda.html`** — misma memoria técnica, maquetada para impresión. Para generar el PDF a adjuntar:
  1. Abrir el archivo en un navegador (doble clic, o clic derecho → Abrir con → Chrome/Edge).
  2. `Ctrl+P` → Destino: **Guardar como PDF** → márgenes por defecto → Guardar.

## Coherencia con el repositorio

El nombre, la versión (1.0.0) y los lenguajes declarados en este anexo coinciden con `CHANGELOG.md` (entrada `[1.0.0] - 2026-09-22`) y con el código fuente referenciado por ruta exacta en la sección 3 del anexo. Si el repositorio cambia de versión o de módulos principales, este anexo debe actualizarse antes de volver a presentarlo, para que lo declarado siga coincidiendo con lo adjunto.

## Autoría

- Esteban Eduardo Escárraga Túquerres.
- Johan Felipe Ordoñez Jenoy.
- Dayerli Katerine Tamayo Solarte.
