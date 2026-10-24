## ETL vs ELT
- **ETL**: se extrae, se transforma en una herramienta aparte (DataStage, PowerCenter) y se carga el resultado.
- **ELT**: se extrae, se carga el dato tal cual en la base o el lake, y se transforma **dentro** de ella con SQL.
- dbt es la **T de ELT**: no tiene motor propio, le pasa el SQL a la base (DuckDB, Snowflake, Athena).

## dbt (data build tool)
`pip install dbt-duckdb` (instala dbt Core y el conector para DuckDB).
- Cada modelo es un archivo `.sql` con **solo un `SELECT`**. dbt crea la tabla o vista.
- Ordena la ejecución según las dependencias entre modelos (deducidas de `ref()`).
- Ejecuta pruebas de calidad declaradas en YAML.
- Genera documentación y linaje.
- No llama a APIs ni mueve datos entre sistemas. No decide cuándo correr: eso es de un orquestador (Airflow, GitHub Actions).
- Existe dbt v2 (motor Fusion, en Rust). En este proyecto se usa dbt Core v1, más estable con DuckDB.

## Comandos dbt
- `dbt --version` -> versión de dbt Core y del plugin.
- `dbt init nombre` -> crea el proyecto y su estructura de carpetas (no se crea a mano).
- `dbt debug` -> prueba la conexión. Debe terminar en "All checks passed!".
- `dbt run` -> ejecuta los modelos (crea tablas y vistas).
- `dbt test` -> ejecuta las pruebas de calidad.
- Se ejecutan desde la carpeta del proyecto dbt.

## Estructura del proyecto dbt
- `dbt_project.yml` -> configuración del proyecto.
- `models/` -> los modelos `.sql` y los `.yml` de fuentes y pruebas.
- `models/staging/` -> un modelo por fuente, prepara los datos (renombra, filtra).
- `models/marts/` -> tablas finales: cruces y reglas de negocio (gold).
- `target/`, `logs/`, `dbt_packages/` -> generadas al ejecutar. No se suben a Git.
- `target/compiled/` -> el SQL final que se ejecutó, sin Jinja.

## profiles.yml
Está en `C:\Users\<usuario>\.dbt\profiles.yml`, **fuera del repositorio** (en proyectos reales lleva contraseñas).
- Define la conexión por ambiente (`dev`, `prod`...). `target:` indica el ambiente por defecto.
- Para DuckDB: `type: duckdb` y `path:` al archivo `.duckdb` (se crea si no existe).

## Jinja
Lenguaje de plantillas (implementado en Python) que dbt usa dentro del SQL. Misma idea que `<?php ?>`, `#parametro#` de DataStage o `&variable` de SQL*Plus, pero reducido a propósito.
- `{{ ... }}` -> se reemplaza por un valor.
- `{% ... %}` -> lógica (`if`, `for`).
- La base de datos nunca ve el Jinja: dbt lo compila a SQL puro antes de ejecutar.

## source() y ref()
- `{{ source('silver', 'uf') }}` -> datos que vienen de afuera, declarados en un `sources.yml`.
- `{{ ref('stg_uf') }}` -> otro modelo de dbt. Con esto dbt sabe el orden de ejecución.

## Sources con archivos Parquet (dbt-duckdb)
```yaml
sources:
  - name: silver
    meta:
      external_location: "{{ env_var('RUTA_DATOS', '../data') }}/silver/indicador={name}/**/*.parquet"
    tables:
      - name: uf
      - name: dolar
```
- `{name}` se reemplaza por el nombre de cada tabla.
- `env_var('VARIABLE', 'valor_por_defecto')` -> lee una variable de entorno; si no existe, usa el valor por defecto. Evita rutas completas en archivos versionados.

## Materializaciones
Se configuran con `{{ config(materialized='table') }}` al inicio del modelo, o por carpeta en `dbt_project.yml`.
- `view` (por defecto) -> consulta guardada; no copia datos.
- `table` -> `CREATE TABLE AS SELECT`; los datos quedan escritos.
- `incremental` -> solo inserta o actualiza lo nuevo.
- `ephemeral` -> no crea nada; se inserta como CTE en otros modelos.
- Convención común: staging en `view`, marts en `table`.

## Pruebas de calidad
Se declaran en un `.yml` junto a los modelos, con la clave `data_tests:` (con `s`; sin ella, dbt la ignora sin avisar).
```yaml
models:
  - name: uf_en_dolares
    columns:
      - name: fecha
        data_tests:
          - not_null
          - unique
```
- Incluidas: `not_null`, `unique`, `accepted_values`, `relationships`.
- Cada prueba es una consulta que debe devolver **cero filas**.
- `severity: warn` -> la prueba avisa pero no detiene el proceso (caso conocido y aceptado):
```yaml
          - not_null:
              config:
                severity: warn
```
- Niveles: genéricas incluidas, paquetes (dbt-utils, dbt-expectations), pruebas singulares (`.sql` en `tests/`) y genéricas propias (macros).

## YAML
- `clave: valor` (con espacio después de los dos puntos).
- La sangría define la estructura. **Solo espacios, nunca tabulaciones.**
- `-` al inicio indica un elemento de lista.
- `version: 2` en los `.yml` de dbt es la versión del formato del archivo, no del proyecto.

## DuckDB desde Python
- `duckdb.sql(...)` -> trabaja en memoria.
- `duckdb.connect(ruta, read_only=True)` -> abre un archivo `.duckdb`. `read_only` evita bloquearlo.
- Solo un programa a la vez puede escribir en el archivo: cerrar otras conexiones antes de `dbt run`.
- `CTE`: un solo `WITH`, CTE separados por comas, sin coma después del último.
- `ASOF LEFT JOIN ... ON a.fecha >= b.fecha` -> une cada fila con la fecha más cercana anterior o igual. No existe en Oracle, SQL Server ni Athena.
