# Conceptos de aprendizaje y proyectos en Pyton

## Funciones, parametros y conceptos mientras desarrollo proyectos.

## `pathlib.Path`
`from pathlib import Path`
Maneja rutas de archivos y carpetas. Funciona igual en Windows y Linux.
- `/` une partes de una ruta: `Path("data") / "bronze" / "uf.json"`
- `.parent` -> la carpeta que contiene el archivo.
- `.mkdir(parents=True, exist_ok=True)` -> crea la carpeta. `parents=True` crea también las de arriba si faltan; `exist_ok=True` no da error si ya existe.
- Ruta relativa (`"../data"`) -> se calcula desde donde está parada la terminal, NO desde el archivo `.py`.

## `Path(__file__).resolve().parent.parent`
- `__file__` -> Python guarda automáticamente la ruta del archivo que se está ejecutando. Siempre existe.
- `.resolve()` -> la convierte en ruta completa (desde `C:\`).
- `.parent` -> sube un nivel. Dos veces: de `src/extraer.py` a la raíz del proyecto.
- Sirve para que la ruta funcione desde cualquier carpeta donde se ejecute.

## f-strings
`f"texto {variable}"` -> mete variables dentro del texto.
No necesita `str()`: convierte el valor a texto solo.

## `requests`
`pip install requests` (no viene con Python).
- `requests.get(url, headers=headers, timeout=10)` -> pide datos a una URL.
- `timeout=10` -> no espera más de 10 segundos. Sin esto, puede quedarse esperando para siempre.
- `headers` -> diccionario con información extra de la petición. `User-Agent` dice quién pide; algunos servidores bloquean el que trae `requests` por defecto.
- `resp.status_code` -> código de respuesta. 200 = OK.
- `resp.raise_for_status()` -> lanza un error si el código no es de éxito (2xx).
- `resp.json()` -> convierte la respuesta JSON en diccionario o lista de Python.

## `json`
`import json` (viene con Python).
- `json.dump(datos, archivo)` -> escribe datos de Python en un archivo JSON.
- `json.load(archivo)` -> lee un archivo JSON y lo convierte a Python.
- `ensure_ascii=False` -> mantiene tildes y ñ tal cual.
- `indent=2` -> guarda el JSON con sangría, legible.
- Para explorar: `datos.keys()` muestra las claves del primer nivel.

## `with open(...) as archivo`
`with open(ruta, "w", encoding="utf-8") as archivo:`
- `"w"` escribe (crea o reemplaza), `"r"` lee, `"a"` agrega al final.
- `as archivo` -> nombre del archivo abierto, para usarlo adentro.
- Al salir del bloque `with`, el archivo se cierra solo.
- `open` crea el archivo, pero NO las carpetas: deben existir antes.

## `try / except`
Captura errores para que el programa no se caiga.
```python
try:
    resp = requests.get(url, timeout=10)
except requests.exceptions.RequestException as error:
    print("Falló:", error)
```
`RequestException` agrupa todos los errores de `requests` (conexión, timeout, SSL).

## `time.sleep(segundos)`
`import time` -> pausa el programa. Se usa entre reintentos: reintentar de inmediato no sirve si el servidor está limitando.

## `if __name__ == "__main__":`
El código dentro solo corre si ejecutas ese archivo directamente.
Si otro archivo lo importa, no corre. Sirve para que las funciones se puedan reutilizar.

