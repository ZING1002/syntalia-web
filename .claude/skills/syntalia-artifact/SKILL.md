---
name: syntalia-artifact
description: Crea y publica cualquier artefacto (documento de decisión, plan, propuesta, resumen técnico) como una página con pestañas y la identidad visual de Syntalia (navy, crema y oro; Poppins y Raleway), con diagramas, modelo de datos, preguntas abiertas y comentarios en vivo. Se guarda en doc/artefactos/ del repositorio y se publica con la herramienta Artifact. Úsala para todos los artefactos que pida Joselu.
argument-hint: <nombre-del-documento>
user-invocable: true
---

# Artefacto Syntalia

Convierte un plan, una propuesta o un análisis en una página que se revisa en unos dos minutos y donde se puede comentar cada punto. Toma como referencia la estructura de la skill `dev-artifact` de Atauri-Labs/otto-platform (pestañas, protocolo de comentarios), pero con la identidad de Syntalia. No modifica nada de ese repositorio.

## Pasos

1. Comprueba que la herramienta Artifact está disponible. Si no lo está, construye la página igualmente y di que queda sin publicar y por qué.
2. Reúne el contenido: decisiones tomadas, diagramas necesarios, cambios de datos si los hay y lo que queda pendiente de decidir.
3. Copia `references/template.html` a `doc/artefactos/<nombre-del-documento>/overview.html`, en la raíz del repositorio, y rellénalo. Mantén su estructura, sus clases y sus scripts. Rellena los atributos de `#review`: `data-spec` (el nombre del documento), `data-artifact` (la URL), `data-version` (v1, v2…) y `data-round` (empieza en 1 y sube en cada nueva publicación). Pon `data-item`, `data-kind` y `data-title` en cada decisión, pregunta, diagrama y tabla de datos (ver "Comentarios"). En una revisión, conserva los IDs existentes.
4. Antes de declarar capacidades, carga la skill `artifact-capabilities`.
5. Publica con la herramienta Artifact:
   - `file_path`: ese archivo;
   - `icon`: una palabra genérica, la misma en cada publicación del mismo documento;
   - `description`: el objetivo en una línea;
   - `label`: la versión (igual que `data-version`);
   - `capabilities`: `{"db": {"rules": [{"path": "", "read": "view", "write": "admin"}]}, "comments": {}, "downloads": true}`.
   En una revisión, publica en la misma URL: la misma ruta en la misma sesión, o pasa `url` desde otra sesión. Nunca crees un segundo artefacto para el mismo documento: así se conservan los comentarios.
6. Tras la primera publicación, haz una sola comprobación: un `list` de la colección `review` con ArtifactData. Debe responder, aunque esté vacía.
7. Actualiza el índice `doc/artefactos/README.md` con el artefacto, su carpeta, su enlace, su versión y su estado. Si exportas un PDF, guárdalo en la misma carpeta del artefacto.
8. Devuelve el enlace. Es privado hasta que el dueño lo comparte desde el menú Compartir de la página.

## Forma de trabajo: iterar sobre el mismo artefacto

Joselu trabaja iterando sobre el mismo artefacto: cada ronda de cambios se aplica en `doc/artefactos/<nombre>/overview.html`, se suben `data-version` y `data-round`, se vuelve a publicar en la misma URL y se actualiza la fila del índice. Para cada ronda, usa también las skills que correspondan al contenido: por ejemplo `artifact-capabilities` para las capacidades de la página, `dataviz` para gráficos y `pdf` para exportar. Solo se crea un artefacto nuevo cuando es un tema distinto.

## Identidad Syntalia (obligatoria)

Sale del sistema de diseño "Syntalia" (https://claude.ai/artifact/HNLzPGLC5iQH2Dr3JgYhK2). Si cambian sus tokens, actualiza la plantilla.

- **Navy siempre como fondo** (`#021557`). Los bloques más oscuros, el hero y las cabeceras de tabla usan `navy-night` (`#010823`), y los campos de texto `navy-edge` (`#00040F`). Nunca un fondo claro.
- **Crema para el texto** (`#F5F2E9`), y crema al 72 % para el texto secundario. Las superficies y las líneas finas son crema con poca opacidad. No se añade ningún otro color.
- **El oro es el único acento** (`#D4AF37`) y solo va en lo que brilla: la línea de 88×3 px bajo el título, la pestaña activa, los nodos clave de los diagramas, la etiqueta "Recomendado", las casillas y el botón Aprobar. Nunca como fondo detrás de texto, salvo en el botón Aprobar.
- **Tipografía:** Poppins (600 y 700, con espaciado) para los títulos, las etiquetas en mayúsculas y los botones. Raleway para todo lo demás, también los diagramas y los datos. Ninguna otra fuente.
- **Precisión:** esquinas rectas en bloques y tarjetas (radio 0) y 4 px solo en controles pequeños.
- **Tono:** ingeniería de precisión, no magia. Sí: líneas rectas, encajes secos, luz tenue y dirigida. No: purpurina, destellos en estrella, humo, partículas de colores ni degradados de arcoíris.
- **Cabecera:** barra con la palabra SYNTALIA (Poppins 700, espaciado amplio) precedida de una línea dorada. El logo oficial (la S de dos trazos) solo existe en PNG de 345 px. No se redibuja: cuando haya un SVG oficial, sustituye la palabra por él.
- **Hero:** banda `navy-night` a todo el ancho, con viñeteado hacia `navy-edge`, un contraluz crema suave (≈13 %) y estelas doradas rectas que convergen en un anillo dorado (las piezas que encajan). Junto al texto se atenúa hasta el 12 % para que siempre se lea. Con `prefers-reduced-motion` se dibuja una vez y no se mueve, y se pausa fuera de pantalla. Todo es canvas propio, sin imágenes ni librerías.
- **Una sola apariencia oscura**, a propósito: `color-scheme: dark`, sin variante clara.

## Pestañas (accesibles: `role="tablist"`, flechas del teclado, estado en el hash de la URL)

Adapta los nombres al contenido, manteniendo este orden lógico:

1. **Decisiones** (se abre por defecto): una tarjeta por decisión, con qué se elige, por qué y qué descarta ("Descarta"). Debajo, tablas de apoyo: riesgos, criterios o cambios.
2. **Diagramas**: SVG en línea. Usa `.box` para los elementos, `.hot` (borde dorado) para lo nuevo o clave, `.grp` (discontinuo) para lo que no cambia, `.edge` para las flechas y `.edge.gen` para un flujo que falla o que es especial. Ancho mínimo de 700 px dentro de `.scroll`. Añade un diagrama de flujo cuando lo haya y uno de entidades cuando cambien los datos.
3. **Modelo de datos**: solo si cambia la base de datos; si no, quita la pestaña, su panel y su id en el script. Una tabla separada por cada tabla de la base de datos, indicando si es nueva, modificada o eliminada, con las columnas columna, cambio, tipo y de dónde sale el dato.
4. **Preguntas abiertas**: cada una con su respuesta recomendada. Al final, los criterios de aceptación como lista de casillas.

Que se pueda leer en unos dos minutos: una línea por decisión y nada de copiar documentos enteros.

## Comentarios

La página es interactiva: cada elemento tiene su campo de comentario, y arriba hay una barra con Aprobar (el único botón dorado), Enviar comentarios a Claude y Exportar comentarios, además de un comentario general. Los comentarios se guardan en la base del artefacto (`review/round-<n>`), y Enviar avisa a la sesión que publicó. Los IDs (D, Q, G, M), los mensajes `SPEC-FEEDBACK` y el formato de exportación están en `references/feedback.md`. Mantén ese protocolo sincronizado con el script de la plantilla. Los textos de la interfaz van en español; los mensajes del protocolo no se traducen.

## Codificación

La primera línea de la plantilla es `<meta charset="utf-8">`: no la quites. Escribe los caracteres directamente o como entidades HTML, nunca como escapes `\uXXXX`. Antes de publicar, comprueba que el archivo no contiene `Ã`, `Â`, `â€`, `\uXXXX` ni `�`.

## PDF (si se pide)

Genera el PDF con Edge en modo headless (`--headless --print-to-pdf --no-pdf-header-footer`) a partir de una copia completa del HTML (con doctype). Revisa cada página renderizada antes de entregarlo.
