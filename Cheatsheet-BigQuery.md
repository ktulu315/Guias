# Cheatsheet Rápido de BigQuery

Referencia rápida en orden progresivo de necesidad. BigQuery usa **Standard SQL** (ANSI compatible). Las funciones y sintaxis son específicas de BigQuery a menos que se indique lo contrario.

## 1. Conexión y Configuración

- `bq query --use_legacy_sql=false "SELECT 1"` → Ejecutar consulta desde CLI. `--use_legacy_sql=false` activa Standard SQL (obligatorio hoy).
- `bq ls` → Listar datasets del proyecto por defecto
- `bq ls mi_proyecto:mi_dataset` → Listar tablas dentro de un dataset
- `bq show mi_proyecto:mi_dataset.mi_tabla` → Ver metadatos de una tabla (schema, particionamiento, tamaño)
- `bq mk mi_dataset` → Crear un dataset nuevo
- `bq rm -r mi_dataset` → Eliminar dataset (y tablas dentro)
- `bq load --source_format=CSV mi_dataset.mi_tabla archivo.csv schema.json` → Cargar datos desde CSV/JSON/Parquet/Avro
- `bq extract --destination_format=CSV mi_proyecto:dataset.tabla gs://bucket/export-*.csv` → Exportar tabla a GCS
- `bq query --dry_run "SELECT ..."` → Estimar bytes procesados sin ejecutar
- `#standardSQL` o `#legacySQL` → Comentario al inicio de la consulta para forzar el dialecto

> **Cuota:** BigQuery cobra por **bytes procesados** (on-demand) o por **slots contratados** (flat-rate). Usar `--dry_run` o `INFORMATION_SCHEMA` para estimar costo antes de ejecutar.

## 2. Consultas Básicas

- ``SELECT * FROM `proyecto.dataset.tabla`` → Seleccionar todo (usar **backticks** para rutas completas con proyecto)
- `SELECT col1, col2 FROM dataset.tabla` → Columnas específicas (el proyecto actual se asume por defecto)
- `SELECT * EXCEPT (col_excluida) FROM tabla` → Todas las columnas **menos** una (evita listar 50 columnas)
- `SELECT * REPLACE (UPPER(nombre) AS nombre) FROM tabla` → Todas las columnas pero reemplazando una con transformación
- `SELECT columna AS alias FROM tabla` → Alias con `AS`
- `SELECT DISTINCT col FROM tabla` → Valores únicos (puede ser costoso en tablas grandes)
- `SELECT * FROM tabla WHERE condicion` → Filtrar filas
- `SELECT * FROM tabla ORDER BY col DESC LIMIT 10` → Ordenar y limitar
- `SELECT * FROM tabla LIMIT 10 OFFSET 20` → Paginación (evitar en tablas grandes — mejor usar `WHERE` con clave)
- `SELECT col FROM tabla ORDER BY col DESC LIMIT 10 OFFSET 0` → Con `ORDER BY`, LIMIT es obligatorio (a partir de 4000+ filas sin LIMIT puede dar error de recursos)

> **Proyecto cualificador:** `proyecto.dataset.tabla` (con backticks si el proyecto contiene guiones). Sin proyecto, usa el proyecto por defecto del entorno.

## 3. Tipos de Datos

- `INT64` → Entero de 64 bits. Es el tipo numérico principal para enteros
- `FLOAT64` → Decimal de doble precisión (IEEE 754). Para cálculos con decimales grandes
- `NUMERIC` → Decimal exacto de hasta 38 dígitos (9 decimales). **Preferir sobre FLOAT64 para dinero**
- `BIGNUMERIC` → Decimal exacto de hasta 76 dígitos (38 decimales). Para precisiones extremas
- `STRING` → Texto en UTF-8. Se escribe con comillas simples o dobles
- `BOOL` → `TRUE` o `FALSE` (no `1`/`0`)
- `BYTES` → Datos binarios. Se escriben como `B"..."` o `FROM_BASE64(...)`
- `DATE` → Fecha sin hora: `DATE '2026-05-19'`
- `DATETIME` → Fecha con hora (sin zona horaria): `DATETIME '2026-05-19 14:30:00'`
- `TIMESTAMP` → Momento exacto en UTC con zona horaria: `TIMESTAMP '2026-05-19 14:30:00+02'`
- `TIME` → Hora sin fecha: `TIME '14:30:00'`
- `ARRAY<TIPO>` → Lista ordenada de elementos del mismo tipo. Se escribe con corchetes: `[1, 2, 3]`
- `STRUCT<col TIPO, ...>` → Grupo de campos anidados. Similar a una fila dentro de una columna
- `JSON` → Tipo nativo para JSON (desde 2022). Permite `JSON_QUERY`, `JSON_VALUE` sin extraer como string
- `GEOGRAPHY` → Datos espaciales (WKT, GeoJSON). Requiere licencia de BigQuery GIS

> **ARRAY vs STRUCT:** Un `STRUCT` es una **fila** con múltiples campos (como un objeto). Un `ARRAY` es una **lista** de elementos del mismo tipo. Puedes tener `ARRAY<STRUCT<...>>` para datos completamente anidados.

## 4. Filtros y Condiciones

- `WHERE col IN (1, 2, 3)` → Lista de valores. BigQuery lo optimiza bien
- `WHERE col NOT IN (1, 2, 3)` → Excluir valores. **Cuidado con NULLs**: `NOT IN` con subconsulta que devuelve NULL da 0 resultados
- `WHERE col BETWEEN 10 AND 20` → Rango inclusivo
- `WHERE col IS NULL` → Valores nulos. `NULL = NULL` es falso en SQL
- `WHERE col IS NOT NULL` → Valores no nulos
- `WHERE col LIKE 'prefix%'` → Empieza con (usa índices si la tabla está clusterizada)
- `WHERE col LIKE '%infijo%'` → Contiene (escaneo completo — costoso en tablas grandes)
- `WHERE col = 'texto'` → Igualdad exacta. Preferir sobre LIKE para búsquedas exactas
- `WHERE STARTS_WITH(col, 'prefix')` → Función equivalente a `LIKE 'prefix%'`. Más explícita
- `WHERE col IN UNNEST([1, 2, 3])` → Comparar contra un array (útil si el array es dinámico)
- `WHERE EXISTS (SELECT 1 FROM otra WHERE cond)` → Subconsulta correlacionada. BigQuery la ejecuta eficientemente

> **LIKE vs STARTS_WITH:** Ambas son equivalentes en rendimiento. `STARTS_WITH` es más legible. Para búsqueda de texto completo, considera `SEARCH()` (disponible desde 2024).

## 5. Agregación

- `SELECT COUNT(*) FROM tabla` → Contar filas. **No usa NULLs para decidir** — cuenta todo
- `SELECT COUNT(col) FROM tabla` → Contar filas donde `col` NO es NULL
- `SELECT COUNT(DISTINCT col) FROM tabla` → Contar valores únicos. Preciso pero costoso en tablas grandes. Para estimaciones rápidas, usar `APPROX_COUNT_DISTINCT`
- `SELECT SUM(col), AVG(col), MIN(col), MAX(col) FROM tabla` → Agregados estándar. `AVG` ignora NULLs
- `SELECT COUNTIF(condicion) FROM tabla` → Contar filas que cumplen una condición (equivalente a `SUM(CASE WHEN ...)` pero más legible)
- `SELECT APPROX_COUNT_DISTINCT(col) FROM tabla` → Aproximación de valores únicos (1-2% de error). **Mucho más rápido y barato que COUNT(DISTINCT)** en tablas grandes
- `SELECT ARRAY_AGG(col LIMIT 10) FROM tabla` → Agrupar valores en un array. Con `LIMIT` para evitar arrays gigantes
- `SELECT STRING_AGG(col, ', ' ORDER BY col) FROM tabla` → Concatenar strings con separador. Opcionalmente ordenado
- `SELECT dept, SUM(salario) FROM empleados GROUP BY dept` → Agrupar y agregar por categoría
- `SELECT dept, SUM(salario) FROM empleados GROUP BY 1` → `GROUP BY 1` = agrupar por la primera columna del SELECT
- `SELECT dept, SUM(salario) total FROM empleados GROUP BY dept HAVING total > 10000` → Filtrar grupos después de agregar (no puedes usar `WHERE` con agregados)
- `SELECT col, ANY_VALUE(col2) FROM tabla GROUP BY col` → Toma **un valor arbitrario** del grupo. Evita tener que agregar columnas no agrupadas. El valor no es determinista (no el primer ni el último)

> **WHERE vs HAVING:** `WHERE` filtra filas **antes** de agrupar. `HAVING` filtra grupos **después** de la agregación. Para filtrar antes de agrupar (más eficiente), usa `WHERE`.

## 6. Window Functions

- `ROW_NUMBER() OVER (PARTITION BY dept ORDER BY salario DESC) AS rn` → Número de fila único por departamento, empezando desde 1. Útil para deduplicar
- `RANK() OVER (ORDER BY salario DESC) AS rk` → Ranking con **empates y saltos** (1, 1, 3). Para tablas de posiciones con empates
- `DENSE_RANK() OVER (ORDER BY salario DESC) AS dr` → Ranking sin saltos (1, 1, 2). Cuando quieres que después de un empate venga el siguiente número
- `LAG(col, 1) OVER (ORDER BY fecha) AS prev` → Valor de la fila **anterior** (offset 1). Para comparar fila actual con la anterior
- `LEAD(col, 1) OVER (ORDER BY fecha) AS next` → Valor de la fila **siguiente** (offset 1). Para comparar fila actual con la siguiente
- `NTILE(4) OVER (ORDER BY salario) AS quartile` → Divide el resultado en N grupos iguales. Para percentiles, cuartiles
- `SUM(col) OVER (PARTITION BY dept) AS total_dept` → Agregado sin colapsar filas. Cada fila mantiene su identidad pero ve el total del grupo
- `SUM(col) OVER (ORDER BY fecha ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS running_total` → **Running total**: suma acumulada desde el inicio hasta la fila actual
- `FIRST_VALUE(col) OVER (PARTITION BY dept ORDER BY fecha) AS primer_valor` → Primer valor del frame. Para comparar cada fila contra el primer elemento

> **Rendimiento:** Las window functions procesan todo el conjunto antes de devolver resultados. En tablas muy grandes (>100M filas), usar `PARTITION BY` para reducir el tamaño del frame. `ROWS BETWEEN UNBOUNDED PRECEDING AND UNBOUNDED FOLLOWING` procesa toda la partición.

## 7. Arrays y UNNEST

- `SELECT [1, 2, 3] AS arr` → Literal array. Todos los elementos deben ser del mismo tipo
- `SELECT GENERATE_ARRAY(1, 10, 2) AS arr` → Generar array del 1 al 10 con paso 2: `[1, 3, 5, 7, 9]`
- `SELECT GENERATE_DATE_ARRAY('2026-01-01', '2026-01-10', INTERVAL 1 DAY) AS fechas` → Array de fechas para rellenos de calendario
- `SELECT ARRAY_LENGTH(arr) AS len` → Longitud del array
- `SELECT * FROM tabla, UNNEST(array_col) AS elemento` → **Aplanar array**: una fila por cada elemento del array. Es un `CROSS JOIN` implícito. BigQuery lo ejecuta como `LEFT JOIN UNNEST` si la tabla no tiene filas con array vacío
- `SELECT * FROM tabla LEFT JOIN UNNEST(array_col) AS elemento` → Igual que el anterior pero **preserva filas** con array NULL o vacío
- `SELECT col, (SELECT STRING_AGG(e, ', ') FROM UNNEST(arr) AS e) FROM tabla` → Subconsulta correlacionada con UNNEST para transformar un array en el SELECT
- `SELECT * FROM tabla WHERE EXISTS (SELECT 1 FROM UNNEST(arr) AS e WHERE e > 5)` → Filtrar filas donde el array contiene un elemento que cumple la condición
- `SELECT col FROM tabla, tabla.array_col AS elemento WHERE elemento > 5` → Filtrar después de aplanar
- `SELECT ARRAY_AGG(DISTINCT col ORDER BY col LIMIT 100) FROM tabla` → Reconstruir array desde filas, con DISTINCT, ORDER y LIMIT

> **UNNEST es caro:** Aplanar arrays grandes multiplica las filas. Si el array tiene 1000 elementos, una fila se convierte en 1000 filas. Siempre filtrar lo más pronto posible.

## 8. STRUCTs y Datos Anidados

- `SELECT STRUCT(1 AS id, 'Alice' AS nombre) AS persona` → Literal STRUCT. Los campos se nombran con `AS`
- `SELECT persona.id, persona.nombre FROM tabla` → Acceder a campos de un STRUCT con **notación punto**
- `SELECT col.* FROM tabla` → Expandir campos del STRUCT al mismo nivel
- `SELECT p.id, p.nombre FROM tabla, UNNEST(personas) AS p` → Aplanar un `ARRAY<STRUCT<...>>`. Cada STRUCT del array se convierte en una fila
- `SELECT * EXCEPT (campo_anidado), campo_anidado.* FROM tabla` → Expandir STRUCT anidado manteniendo columnas planas
- `SELECT * REPLACE (UPPER(nombre) AS nombre) FROM tabla` → Reemplazar una columna por su transformación en el resultado
- `SELECT ARRAY_AGG(STRUCT(id, nombre)) FROM tabla` → Agrupar filas en un array de STRUCTs. Útil para datos jerárquicos
- `SELECT ARRAY_AGG(DISTINCT STRUCT(id, nombre)) FROM tabla` → Agrupar evitando duplicados en el array

> **Modelado anidado:** BigQuery es más eficiente con datos **anidados desnormalizados** (arrays de structs) que con tablas normalizadas con joins. Reduce el escaneo porque los datos relacionados están en la misma fila. Una tabla con `ARRAY<STRUCT<...>>` suele ser más rápida y barata que normalizar con 3 tablas y joins.

## 9. Fechas y Timestamps

- `SELECT DATE '2026-05-19' AS hoy` → Literal DATE (formato ISO 8601: YYYY-MM-DD)
- `SELECT TIMESTAMP '2026-05-19 14:30:00+02'` → Literal TIMESTAMP con zona horaria (se almacena en UTC internamente)
- `SELECT CURRENT_DATE() AS hoy` → Fecha actual del servidor
- `SELECT CURRENT_TIMESTAMP() AS ahora` → Timestamp actual UTC
- `SELECT DATE_ADD(fecha, INTERVAL 7 DAY) AS una_semana` → Sumar días/semanas/meses/años
- `SELECT DATE_SUB(fecha, INTERVAL 1 MONTH) AS mes_atras` → Restar intervalo
- `SELECT DATE_DIFF(fecha1, fecha2, DAY) AS dias_diff` → Diferencia entre dos fechas en la unidad especificada (DAY, WEEK, MONTH, QUARTER, YEAR)
- `SELECT EXTRACT(YEAR FROM fecha) AS año` → Extraer año, mes, día, día_de_semana, trimestre de una fecha
- `SELECT EXTRACT(HOUR FROM timestamp) AS hora` → Extraer hora/minuto/segundo de un timestamp
- `SELECT FORMAT_TIMESTAMP('%Y-%m-%d %H:%M:%S', timestamp, 'America/Argentina/Buenos_Aires') AS formateado` → Formatear timestamp con zona horaria. Usar **timezone de IANA**
- `SELECT PARSE_TIMESTAMP('%Y-%m-%d %H:%M:%S', '2026-05-19 14:30:00') AS ts` → Convertir string a timestamp (formato con códigos de strftime)
- `SELECT TIMESTAMP_TRUNC(timestamp, MONTH) AS inicio_mes` → Truncar timestamp a la unidad (comienzo del día, mes, año)
- `SELECT DATETIME_TRUNC(datetime, HOUR) AS inicio_hora` → Truncar datetime a la hora
- `SELECT LAST_DAY(fecha, MONTH) AS fin_mes` → Último día del mes de la fecha dada

> **Zonas horarias:** BigQuery almacena los `TIMESTAMP` en **UTC** y convierte al mostrar. Para operaciones en zona local, usar `DATETIME` en vez de `TIMESTAMP`, o usar `TIMESTAMP` con la zona horaria en las funciones.

## 10. Strings y Regex

- `SELECT CONCAT('Hola', ' ', 'Mundo') AS saludo` → Concatenar strings. Si algún argumento es NULL, el resultado es NULL
- `SELECT CONCAT(col1, ' - ', col2) FROM tabla` → Concatenar columnas con separador
- `SELECT SUBSTR('Hola Mundo', 1, 4) AS sub` → Substring (posición 1-indexada). Extrae desde posición 1, 4 caracteres → 'Hola'
- `SELECT REPLACE('Hola Mundo', 'Mundo', 'GCP')` → Reemplazar todas las ocurrencias
- `SELECT TRIM('  Hola  ') AS limpio` → Eliminar espacios al inicio y final
- `SELECT LTRIM('  Hola')`, `RTRIM('Hola  ')` → Eliminar espacios solo a izquierda o derecha
- `SELECT UPPER('hola')`, `LOWER('HOLA')` → Mayúsculas y minúsculas
- `SELECT STARTS_WITH(col, 'prefix')` → ¿Empieza con?
- `SELECT ENDS_WITH(col, 'suffix')` → ¿Termina con?
- `SELECT REGEXP_EXTRACT(col, r'(\d{4})-(\d{2})') AS año_mes` → Extraer la **primera** coincidencia de un grupo de captura regex
- `SELECT REGEXP_EXTRACT_ALL(col, r'\d+') AS numeros` → Extraer **todas** las coincidencias como un array de strings
- `SELECT REGEXP_REPLACE(col, r'\s+', ' ') AS compacto` → Reemplazar usando regex (contraer espacios múltiples)
- `SELECT SPLIT('a,b,c', ',') AS arr` → Dividir string en array (útil con UNNEST después)
- `SELECT SPLIT(col, ',')[SAFE_OFFSET(1)] AS segundo` → Acceder a un elemento del array resultado por índice (0-indexado). `SAFE_OFFSET` evita error si el índice no existe (devuelve NULL)
- `SELECT LENGTH(col) AS len` → Longitud del string (número de caracteres UTF-8)
- `SELECT SEARCH(col, 'texto')` → Búsqueda de texto full-text (disponible en ediciones Enterprise+). Índice opcional para acelerar

> **Regex en BigQuery:** Usa la sintaxis **re2** (no PCRE). No soporta lookahead/lookbehind ni backreferences. Para reemplazos que necesitan backreferences, usar `REGEXP_REPLACE` con `\\1` para retro-referencias.

## 11. CTEs y Subconsultas

- ``WITH ventas_2026 AS (SELECT * FROM `dataset.ventas` WHERE year = 2026) SELECT * FROM ventas_2026`` → CTE (Common Table Expression): define una subconsulta reutilizable. Equivalente a una vista temporal
- `WITH base AS (SELECT ...), agregada AS (SELECT dept, SUM(x) FROM base GROUP BY dept) SELECT * FROM agregada` → Múltiples CTEs separados por coma. Pueden referenciarse entre sí
- ``WITH RECURSIVE nums AS (SELECT 1 AS n UNION ALL SELECT n + 1 FROM nums WHERE n < 10) SELECT * FROM nums`` → CTE recursivo. Limitado a 500 iteraciones por defecto. Útil para jerarquías, generación de series, árboles
- ``SELECT (SELECT COUNT(*) FROM `dataset.ventas` WHERE year = 2026) AS total`` → Subconsulta escalar en el SELECT (debe devolver una sola fila y columna)
- `SELECT * FROM tabla WHERE col > (SELECT AVG(col) FROM tabla)` → Subconsulta escalar en el WHERE
- `SELECT * FROM tabla WHERE id IN (SELECT id FROM otra WHERE cond)` → Subconsulta con IN. BigQuery lo optimiza como semi-join
- `SELECT * FROM tabla AS t WHERE EXISTS (SELECT 1 FROM otra WHERE otra.id = t.id)` → Subconsulta correlacionada con EXISTS. BigQuery la ejecuta como semi-join (eficiente)
- `SELECT * FROM tabla AS t WHERE NOT EXISTS (SELECT 1 FROM otra WHERE otra.id = t.id)` → Anti-join: filas de tabla que **no** están en otra

> **CTE vs Subconsulta:** Las CTEs son más legibles y pueden referenciarse múltiples veces. Pero BigQuery **no materializa** las CTEs (se in linean como subconsultas). Si usas la misma CTE 3 veces, se ejecuta 3 veces. Para reutilización real, crear una **vista** o tabla temporal.
> **IN vs EXISTS:** En BigQuery, `IN` con subconsulta y `EXISTS` son equivalentes en rendimiento. Preferir el que sea más legible.

## 12. Joins

- `SELECT * FROM a JOIN b ON a.id = b.id` → INNER JOIN: solo filas que coinciden en ambas
- `SELECT * FROM a LEFT JOIN b ON a.id = b.id` → Todas las filas de A + datos de B (NULL si no coincide)
- `SELECT * FROM a RIGHT JOIN b ON a.id = b.id` → Todas las filas de B + datos de A
- `SELECT * FROM a FULL JOIN b ON a.id = b.id` → Todas las filas de ambas (NULL donde no coincide)
- `SELECT * FROM a CROSS JOIN b` → Producto cartesiano (cada fila de A con cada fila de B). **Peligroso** en tablas grandes
- `SELECT * FROM a JOIN b USING (id)` → JOIN usando columna con el mismo nombre en ambas tablas (sintaxis más limpia)
- `SELECT * FROM a NATURAL JOIN b` → JOIN implícito usando todas las columnas con el mismo nombre. **Evitar** — cambios de schema pueden romper la consulta silenciosamente
- `SELECT * FROM a JOIN b ON a.fecha BETWEEN b.inicio AND b.fin` → **JOIN por rango** (no equitativo). BigQuery lo soporta pero puede ser costoso. Considerar particionamiento
- `SELECT * FROM a LEFT JOIN b ON a.id = b.id AND b.activo = TRUE` → Condiciones en el JOIN (no en WHERE) para no perder filas de A después del LEFT JOIN
- `SELECT * FROM a JOIN b ON a.id = b.id AND a.fecha = b.fecha` → JOIN compuesto con múltiples condiciones

> **Broadcast vs Hash Join:** BigQuery elige automáticamente la estrategia. Para tablas pequeñas (<10MB), el optimizador usa **broadcast** (copia la tabla a todos los workers). Para tablas grandes, usa **hash join**. Si una tabla es mucho más pequeña que la otra, el JOIN será eficiente. Si ambas son grandes, el JOIN procesa más datos.

> **Joins con particionamiento:** Si ambas tablas están particionadas por la misma clave y el JOIN es por esa clave, BigQuery puede hacer **pruning** y solo leer las particiones necesarias.

## 13. DML (Data Manipulation Language)

- `INSERT INTO dataset.tabla (id, nombre) VALUES (1, 'Alice'), (2, 'Bob')` → Insertar filas. BigQuery puede insertar hasta 10,000 filas por operación DML
- `INSERT INTO dataset.tabla SELECT * FROM otra WHERE cond` → Insertar desde SELECT (CTAS es equivalente pero más eficiente para tablas nuevas)
- `UPDATE dataset.tabla SET col = valor WHERE condicion` → Actualizar filas. **Siempre** usar WHERE — UPDATE sin WHERE actualiza todas las filas (costoso)
- `UPDATE dataset.tabla SET col = valor FROM otra_tabla WHERE tabla.id = otra.id` → UPDATE con JOIN (actualizar desde otra tabla)
- `DELETE FROM dataset.tabla WHERE condicion` → Eliminar filas. Siempre usar WHERE
- `DELETE FROM dataset.tabla WHERE TRUE` → Vaciar tabla (más barato que DROP + recrear, pero el schema se mantiene)
- `MERGE INTO dataset.tabla AS target USING source AS source ON target.id = source.id WHEN MATCHED THEN UPDATE SET ... WHEN NOT MATCHED THEN INSERT ROW` → **Upsert**: inserta o actualiza según exista la fila (Merge DML). Ideal para SCD Tipo 1
- `TRUNCATE TABLE dataset.tabla` → Vaciar tabla rápidamente (más rápido que DELETE sin WHERE). No afecta el schema
- `CREATE TABLE dataset.tabla_nueva AS SELECT * FROM origen` → **CTAS**: crear tabla con datos. Hereda el schema de la consulta
- `CREATE TABLE dataset.tabla_nueva CLONE dataset.origen` → Crear un **clon** (sin copiar datos, solo metadatos). Escritura diferida
- `CREATE TABLE dataset.tabla_nueva COPY dataset.origen` → Crear una **copia física** de la tabla (cuesta almacenamiento completo)

> **DML en BigQuery:** Las operaciones DML (UPDATE, DELETE, MERGE) son **transaccionales** por fila pero tienen límites de 10,000 operaciones por día en tablas particionadas. Para transformaciones masivas, prefiere **CTAS** (CREATE TABLE AS SELECT) sobre UPDATE/DELETE. CTAS es más barato y no cuenta en el límite de DML.

## 14. Particionamiento y Clustering

- `PARTITION BY DATE(fecha_col)` → Particionar tabla por día (recomendado para tablas con fechas). Hasta 4000 particiones
- `PARTITION BY TIMESTAMP_TRUNC(ts_col, DAY)` → Particionar por timestamp truncado al día
- `PARTITION BY DATETIME_TRUNC(dt_col, MONTH)` → Particionar por mes (para tablas con pocos meses de datos)
- `PARTITION BY RANGE_BUCKET(id, GENERATE_ARRAY(0, 1000000, 100000))` → Particionar por rango numérico (ej: rangos de IDs)
- `CLUSTER BY col1, col2` → Ordenar bloques dentro de particiones. Útil para filtros comunes (ej: país + categoría)
- `CLUSTER BY col1, col2, col3` → Hasta 4 columnas de clustering. El orden importa: la primera columna tiene el mejor prune
- `SELECT * FROM tabla WHERE fecha = '2026-05-19'` → **Partition pruning**: BigQuery solo lee la partición necesaria (mucho más barato)
- `SELECT * FROM tabla WHERE pais = 'AR' AND fecha = '2026-05-19'` → Partition prune + cluster prune: primero por partición, luego por clustering dentro de la partición
- `SELECT * FROM tabla WHERE DATE(fecha_col) = '2026-05-19'` → **No usa** partition pruning si envuelves la columna en una función (comparar `fecha_col = DATE '2026-05-19'`)

> **Partición vs Cluster:** Partición reduce el **escaneo** (navega a la partición correcta). Cluster reduce el **escaneo dentro de la partición** (organiza bloques). Usar **partición por fecha** (siempre que la tabla tenga fecha) + **clustering** por columnas de filtro frecuentes. El clustering se beneficia cuando los datos se insertan en orden (por la partición).
> **Límite:** 4000 particiones por tabla. Si tienes datos diarios por 10 años (3650 días), particionar por **mes** en vez de día.

## 15. Optimización y Buenas Prácticas

- `SELECT col1, col2 FROM tabla` → **Siempre** seleccionar solo las columnas necesarias. `SELECT *` procesa todas las columnas (más caro y lento)
- `SELECT * EXCEPT (col_pesada) FROM tabla` → Alternativa para tablas con muchas columnas donde necesitas casi todas
- `WHERE fecha = DATE '2026-05-19'` → Filtrar por partición (no usar funciones en la columna de partición). `WHERE DATE(fecha) = '2026-05-19'` **no** hace pruning
- `ORDER BY col LIMIT 1000` → **Siempre** acompañar ORDER BY con LIMIT en tablas grandes. Sin LIMIT, BigQuery necesita ordenar todas las filas (requiere un worker para el merge final)
- `ORDER BY col DESC LIMIT 0` → `LIMIT 0` para obtener el schema sin procesar datos (gratuito)
- `SELECT ... WHERE _PARTITIONDATE BETWEEN '2026-01-01' AND '2026-01-31'` → Filtrar por pseudo-columna `_PARTITIONDATE` en tablas particionadas
- `SELECT * FROM dataset.tabla_* WHERE _TABLE_SUFFIX BETWEEN '20260101' AND '20260131'` → **Wildcard tables**: consultar múltiples tablas que coinciden con un patrón (usar `_TABLE_SUFFIX` para filtrar)
- `SELECT * FROM dataset.tabla_* WHERE _TABLE_SUFFIX = '20260519'` → Consultar una tabla específica del wildcard (más barato que `SELECT * FROM dataset.tabla_20260519` si todas las tablas tienen el mismo schema)
- `SELECT column FROM ... WHERE cond GROUP BY column LIMIT 10` → Procesar menos datos temprano (filtrar lo más pronto posible en la consulta)
- `EXPLAIN SELECT ...` → Ver el plan de ejecución. Identificar etapas de **shuffle** y **sort** (las más caras)
- `SELECT query, total_bytes_processed, total_slot_ms FROM region-us.INFORMATION_SCHEMA.JOBS_BY_PROJECT WHERE creation_time > TIMESTAMP_SUB(CURRENT_TIMESTAMP(), INTERVAL 7 DAY)` → Auditar costo de consultas recientes
- `SELECT * FROM region-us.INFORMATION_SCHEMA.TABLE_STORAGE WHERE table_name = 'mi_tabla'` → Ver almacenamiento de una tabla (lógico vs físico, compresión, tiempo de última modificación)
- `CREATE TABLE dataset.tabla OPTIONS(require_partition_filter=true) AS SELECT ...` → **Forzar** que todas las consultas a esta tabla tengan filtro de partición. Evita escaneos completos accidentales
- `SET @@default_time_zone = 'America/Argentina/Buenos_Aires';` → Establecer zona horaria por defecto para la sesión
- `SET @@max_statement_count = 100;` → Controlar límites de la sesión de scripting
- `FOR ... IN (SELECT ...) DO ... END FOR;` → **Scripting**: ejecutar bucles y lógica procedural. Útil para migraciones y ETLs complejos

> **Regla de oro de costo en BigQuery:** El costo es proporcional a los **bytes procesados**. Cada byte que puedes evitar escanear es dinero ahorrado. Las técnicas más importantes por orden de impacto: (1) **Particionamiento** + filtro de partición, (2) **SELECT columnas específicas** en vez de `*`, (3) **Clustering** en columnas de filtro frecuentes, (4) **Pre-agregación** en tablas intermedias, (5) `APPROX_COUNT_DISTINCT` en vez de `COUNT(DISTINCT)`.
