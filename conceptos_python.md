# Conceptos de aprendizaje y proyectos en Pyton

## Funciones, parametros y conceptos mientras desarrollo proyectos.

### Path(__file__).resolve().parent.parent

Path viene en el `from pathlib import path`\
`__file__` -> Python guarda automáticamente en esa variable la ruta del archivo que se está ejecutando. Siempre existe.\
`resolve()` -> la convierte en la ruta completa (desde `C:\`)
`parent` -> sube un nivel desde la ruta resuelta.

