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
- `as archivo` -> nombre del archivo abierto, para usarlo adentro.
- Al salir del bloque `with`, el archivo se cierra solo.
- `open` crea el archivo, pero NO las carpetas: deben existir antes.

### Modos de `open`
- `"r"` -> leer. El archivo debe existir.
- `"w"` -> escribir. **Vacía el archivo en el momento de abrirlo**, aunque después no escriba nada.
- `"a"` -> agregar al final, sin borrar lo anterior.

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

## `json.load` vs `json.dump`
- `json.load(archivo)` -> lee el archivo y devuelve diccionario o lista.
- `json.dump(datos, archivo)` -> escribe los datos en el archivo.
- Error `JSONDecodeError: Expecting value: line 1 column 1` -> el archivo está vacío o no es JSON.

## Errores en tiempo de ejecución
Python revisa los nombres (variables, módulos) **cuando ejecuta la línea**, no al cargar el archivo.
Un `import` olvidado en código que solo corre si hay error no se detecta hasta que ese error ocurre.

## Funciones
- Valor por defecto: `def ordena_df(df, campo, ascendente=True):` -> si no se pasa, usa `True`.
- Comparar con `None`: usar `is None`, no `== None`.

## pandas
`pip install pandas` / `import pandas as pd`
- `pd.DataFrame(lista_de_diccionarios)` -> tabla. Cada diccionario es una fila; cada clave, una columna.
- `pd.` crea o lee tablas (`pd.DataFrame`, `pd.read_parquet`); `df.` trabaja sobre una tabla existente.
- `df.head()` -> primeras 5 filas.
- `df.dtypes` -> tipo de cada columna (plural). `df["col"].dtype` -> tipo de una columna.
- `df["nueva"] = valor` -> crea una columna con el mismo valor en todas las filas.
- `df["col"] = ...` -> reemplaza una columna.
- `df.sort_values("col", ascending=True)` -> ordena. **No modifica `df`**: hay que asignar el resultado.
- `.reset_index(drop=True)` -> renumera las filas desde 0 y descarta el índice viejo.
- Tipo `object` -> la columna guarda objetos de Python (por ejemplo, fechas de `.dt.date`).

## Fechas en pandas
- `pd.to_datetime(df["fecha"], utc=True)` -> texto a fecha y hora. `utc=True` indica que el texto viene en UTC.
- `.dt` -> acceso a funciones de fecha sobre una columna.
- `.dt.tz_convert("America/Santiago")` -> convierte de UTC a hora de Chile.
- `.dt.date` -> deja solo la fecha, sin la hora.
- La `Z` al final de una fecha (`2026-11-09T03:00:00.000Z`) significa UTC.

## Parquet
Formato de archivo columnar y binario, estándar en data engineering. Guarda los tipos de cada columna.
`pip install pyarrow` (pandas lo usa por debajo).
- `df.to_parquet(ruta, index=False)` -> guarda. `index=False` no guarda la numeración de filas.
- `pd.read_parquet(ruta)` -> lee. Las fechas vuelven como `object`, aunque en el archivo son `date`.
- Ver el tipo real dentro del archivo:
```python
import pyarrow.parquet as pq
print(pq.read_schema(ruta))
```

## Arquitectura medallion
- **bronze** -> dato crudo, tal como llega de la fuente.
- **silver** -> limpio, tipado y validado.
- **gold** -> listo para consumir: cruces, agregados, reglas de negocio.
Las capas indican el estado del dato, no la herramienta usada.

## Particionamiento estilo Hive
Carpetas con formato `nombre=valor`: `bronze/indicador=uf/year=2026/`.
Athena, Glue y DuckDB las reconocen como columnas y leen solo las carpetas que piden en el `WHERE`.

## DuckDB
`pip install duckdb` / `import duckdb`
Base de datos analítica que corre dentro del programa, sin servidor. Consultas en SQL.
- `duckdb.sql("consulta").show()` -> ejecuta y muestra el resultado.
- `read_parquet('ruta/**/*.parquet', hive_partitioning=true)` -> lee todos los Parquet de las subcarpetas; las particiones aparecen como columnas.
- Se puede abrir desde DBeaver.
- Athena hace lo mismo sobre S3: lo probado en DuckDB se traslada casi igual.

## Forward fill / LOCF
*Last observation carried forward*: si un día no tiene dato (por ejemplo, el dólar en fin de semana), se usa el último valor disponible.
En DuckDB: `ASOF JOIN` une cada fecha con el valor de la fecha más cercana anterior o igual.

