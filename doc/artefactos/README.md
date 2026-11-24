# Artefactos

Aquí se guardan los artefactos del proyecto: una carpeta por artefacto, con la fuente que se publica (`overview.html`) y, si existen, sus exportaciones (PDF). Se trabaja iterando sobre el mismo artefacto: cada cambio se vuelve a publicar en la misma URL, se sube la versión y se actualiza esta tabla.

Los artefactos se crean con la skill `syntalia-artifact` (`.claude/skills/syntalia-artifact`), que Claude Code carga automáticamente al abrir este repositorio.

| Artefacto | Carpeta | Enlace | Versión | Estado |
| --- | --- | --- | --- | --- |

Los enlaces son privados hasta que se comparten desde el menú Compartir de cada página.

## Cómo iterar

1. Editar `<carpeta>/overview.html`.
2. Subir `data-version` (v2, v3…) y `data-round` en `#review`.
3. Volver a publicar en la misma URL (la herramienta Artifact con `url`).
4. Actualizar la fila de esta tabla.
