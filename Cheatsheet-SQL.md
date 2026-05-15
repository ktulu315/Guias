# Cheatsheet Rápido de SQL

Referencia rápida en orden progresivo de necesidad. Orientado a MySQL / PostgreSQL / SQL Server. Las diferencias importantes se indican.

## 1. Conexión y Exploración

- `mysql -u usuario -p -h host -P 3306` → Conectar a MySQL (puerto por defecto 3306)
- `psql -U usuario -d basedatos -h host` → Conectar a PostgreSQL
- `mysql -u root -p < dump.sql` → Importar SQL directamente desde archivo
- `\c basedatos` → Conectarse a una BD (PostgreSQL)
- `SHOW DATABASES;` → Listar bases de datos (MySQL) / `\l` en PostgreSQL
- `USE basedatos;` → Seleccionar base de datos
- `SHOW TABLES;` → Listar tablas (MySQL) / `\dt` en PostgreSQL
- `DESCRIBE tabla;` → Ver estructura de tabla (MySQL) / `\d tabla` en PostgreSQL
- `SHOW INDEX FROM tabla;` → Ver índices de una tabla (MySQL)

## 2. Consultas Básicas

- `SELECT * FROM tabla;` → Todas las columnas (solo para exploración rápida)
- `SELECT col1, col2 FROM tabla;` → Columnas específicas (preferir en producción)
- `SELECT DISTINCT col FROM tabla;` → Valores únicos (puede ser costoso en tablas grandes)
- `SELECT * FROM tabla WHERE condicion;` → Filtrar filas
- `SELECT * FROM tabla ORDER BY col DESC;` → Ordenar
- `SELECT * FROM tabla LIMIT 10 OFFSET 20;` → Paginación
- `SELECT alias.col FROM tabla AS alias;` → Alias de tabla (mejora legibilidad)
- `SELECT col AS nombre FROM tabla;` → Alias de columna

> **Orden de ejecución:** `FROM` → `WHERE` → `GROUP BY` → `HAVING` → `SELECT` → `ORDER BY` → `LIMIT`. No puedes usar alias del SELECT en el WHERE.

## 3. Filtros y Operadores

- `WHERE col LIKE 'prefix%'` → Empieza con (usa índice)
- `WHERE col LIKE '%infijo%'` → Contiene (NO usa índice — escaneo completo)
- `WHERE col ILIKE '%texto%'` → LIKE sin distinguir mayúsculas (PostgreSQL)
- `WHERE col IN (1, 2, 3)` → Lista de valores
- `WHERE col BETWEEN 10 AND 20` → Rango inclusivo
- `WHERE col IS NULL` → Valores nulos (NO usar `= NULL`)
- `WHERE col IS NOT NULL` → Valores no nulos

> **Rendimiento:** `LIKE '%texto'` nunca usa índices. Para búsqueda de texto, considerar `ILIKE` (PG), `FULLTEXT` (MySQL), o búsqueda tipo `=`. `IN` con valores constantes suele ser eficiente. `IS NULL` es distinto de `col = ''` (vacío no es nulo).

## 4. Agregación y Agrupación

- `SELECT COUNT(*) FROM tabla;` → Contar filas (incluye NULLs)
- `SELECT COUNT(col) FROM tabla;` → Contar filas con valor NO NULL en esa columna
- `SELECT SUM(col), AVG(col), MIN(col), MAX(col) FROM tabla;` → Agregados estándar
- `SELECT dept, COUNT(*) FROM empleados GROUP BY dept;` → Agrupar y contar por categoría
- `SELECT dept, AVG(salario) FROM empleados GROUP BY dept;` → Promedio por grupo
- `SELECT dept, AVG(salario) FROM empleados GROUP BY 1;` → `GROUP BY 1` = agrupar por primera columna del SELECT
- `SELECT col, COUNT(*) FROM tabla GROUP BY col HAVING COUNT(*) > 5;` → Filtrar grupos después de agrupar

> **WHERE vs HAVING:** `WHERE` filtra filas **antes** de agrupar. `HAVING` filtra grupos **después** de la agregación. No puedes usar agregados (`SUM`, `COUNT`) en el WHERE.

## 5. Joins

- `SELECT * FROM a JOIN b ON a.id = b.id;` → INNER JOIN: solo filas que coinciden en ambas
- `SELECT * FROM a LEFT JOIN b ON a.id = b.id;` → Todas las filas de A + datos de B (NULL si no coincide)
- `SELECT * FROM a RIGHT JOIN b ON a.id = b.id;` → Todas las filas de B + datos de A
- `SELECT * FROM a FULL JOIN b ON a.id = b.id;` → Todas las filas de ambas (NULL donde no coincide)
- `SELECT * FROM a CROSS JOIN b;` → Producto cartesiano (cada fila de A con cada fila de B)
- `SELECT * FROM a JOIN b USING (id);` → JOIN cuando la columna se llama igual en ambas tablas
- `SELECT * FROM empleados e JOIN empleados j ON e.jefe_id = j.id;` → SELF JOIN: una tabla consigo misma

> **Decisión:** Usar `INNER JOIN` por defecto. `LEFT JOIN` cuando puedas perder filas. `CROSS JOIN` casi nunca (solo para generar datos de prueba). Siempre prefijar columnas con el alias de tabla (`e.nombre`). `USING` es más limpio que `ON` cuando las columnas tienen el mismo nombre.

## 6. Modificación de Datos (DML)

- `INSERT INTO tabla (col1, col2) VALUES (v1, v2);` → Insertar una fila
- `INSERT INTO tabla (col1, col2) VALUES (v1, v2), (v3, v4);` → Insertar varias filas
- `INSERT INTO copia SELECT * FROM original;` → Insertar resultado de un SELECT
- `INSERT INTO tabla VALUES (v1, v2) ON CONFLICT (id) DO UPDATE SET col = EXCLUDED.col;` → Upsert (PostgreSQL)
- `INSERT INTO tabla VALUES (v1, v2) ON DUPLICATE KEY UPDATE col = VALUES(col);` → Upsert (MySQL)
- `UPDATE tabla SET col = valor WHERE condicion;` → Actualizar filas (SIEMPRE con WHERE)
- `UPDATE tabla SET col = subquery FROM otra WHERE ...;` → UPDATE con JOIN (PostgreSQL)
- `DELETE FROM tabla WHERE condicion;` → Eliminar filas (SIEMPRE con WHERE)
- `TRUNCATE TABLE tabla;` → Vaciar tabla entera (rápido, no usa transacción por fila)
- `DELETE FROM tabla;` → Vaciar tabla (lento, usa transacción, dispara triggers)
- `RETURNING *;` → Devolver filas afectadas (PostgreSQL — al final de INSERT, UPDATE, DELETE)

> **Advertencia:** `UPDATE` o `DELETE` sin `WHERE` afecta TODAS las filas. Siempre hacer `SELECT` primero con el mismo WHERE para verificar. `TRUNCATE` no se puede deshacer en la misma transacción en MySQL (pero sí en PG).

## 7. Estructura de Tablas (DDL)

- `CREATE TABLE tabla (id SERIAL PRIMARY KEY, nombre VARCHAR(100) NOT NULL, email TEXT UNIQUE);` → Tabla completa con restricciones
- `CREATE TABLE copia AS SELECT * FROM original;` → Crear tabla a partir de una consulta (sin índices ni constraints)
- `CREATE TABLE copia (LIKE original INCLUDING ALL);` → Clonar estructura con índices (PostgreSQL)
- `ALTER TABLE tabla ADD COLUMN col tipo;` → Agregar columna
- `ALTER TABLE tabla DROP COLUMN col;` → Eliminar columna
- `ALTER TABLE tabla ALTER COLUMN col SET NOT NULL;` → Agregar restricción NOT NULL
- `ALTER TABLE tabla ADD CONSTRAINT fk_name FOREIGN KEY (col) REFERENCES otra(id) ON DELETE CASCADE;` → Clave foránea
- `DROP TABLE IF EXISTS tabla;` → Eliminar tabla si existe (evita error)
- `CREATE INDEX idx_apellido ON empleados (apellido);` → Índice simple en columna
- `CREATE INDEX idx_nombre_apellido ON empleados (nombre, apellido);` → Índice compuesto (orden importa)
- `CREATE UNIQUE INDEX idx_email ON usuarios (email);` → Índice único (como UNIQUE constraint)
- `DROP INDEX IF EXISTS idx_name;` → Eliminar índice si existe

> **Índices compuestos:** El orden de las columnas importa. `(nombre, apellido)` sirve para búsquedas por `nombre`, pero NO para búsquedas solo por `apellido` — aplica la regla de "leftmost prefix".

## 8. Funciones Útiles

- `SELECT UPPER(col), LOWER(col), INITCAP(col) FROM tabla;` → Transformar texto
- `SELECT LENGTH(col) FROM tabla;` → Longitud (caracteres, no bytes)
- `SELECT CONCAT(col1, ' ', col2) FROM tabla;` → Concatenar (MySQL/PG) / `col1 + ' ' + col2` en SQL Server
- `SELECT SUBSTRING(col FROM 1 FOR 3) FROM tabla;` → Extraer parte de texto
- `SELECT TRIM(col), LTRIM(col), RTRIM(col) FROM tabla;` → Quitar espacios
- `SELECT REPLACE(col, 'old', 'new') FROM tabla;` → Reemplazar subcadena
- `SELECT COALESCE(col, 'default') FROM tabla;` → Primer valor NO NULL (mejor que IFNULL)
- `SELECT NULLIF(a, b) FROM tabla;` → NULL si a = b, si no a
- `SELECT CASE WHEN cond THEN 'x' WHEN cond2 THEN 'y' ELSE 'z' END FROM tabla;` → Condicional en consulta
- `SELECT NOW(), CURRENT_DATE, CURRENT_TIME;` → Fecha/hora actual (NOW() es estable dentro de una transacción en PG)
- `SELECT DATE(col), EXTRACT(YEAR FROM col), EXTRACT(MONTH FROM col) FROM tabla;` → Extraer partes de fecha
- `SELECT AGE(fin, inicio) FROM tabla;` → Diferencia como intervalo (PostgreSQL)
- `SELECT DATEDIFF(fin, inicio) FROM tabla;` → Diferencia en días (MySQL) / `DATE_PART('day', fin - inicio)` en PG
- `SELECT TO_CHAR(col, 'YYYY-MM-DD') FROM tabla;` → Formatear fecha como texto (PostgreSQL)
- `SELECT STRING_AGG(nombre, ', ') FROM empleados;` → Concatenar filas en una sola string (PG) / `GROUP_CONCAT` en MySQL
- `SELECT CAST(col AS tipo) FROM tabla;` → Convertir tipo de dato / `col::tipo` en PostgreSQL

> **COALESCE vs CASE:** `COALESCE` es más limpio para "primer valor no nulo". Usar `CASE` para lógica condicional más compleja. `COALESCE` acepta múltiples argumentos y devuelve el primero no NULL.

## 9. Subconsultas y CTEs

- `SELECT * FROM tabla WHERE id IN (SELECT id FROM otra);` → Subconsulta en WHERE (cuidado con NULLs en IN)
- `SELECT * FROM tabla WHERE EXISTS (SELECT 1 FROM otra WHERE otra.id = tabla.id);` → EXISTS (más eficiente que IN con subconsultas grandes)
- `SELECT *, (SELECT MAX(precio) FROM productos) AS max FROM ventas;` → Subconsulta escalar en SELECT
- `SELECT sub.* FROM (SELECT * FROM tabla WHERE condicion) AS sub;` → Subconsulta en FROM (derived table)
- `WITH ventas_altas AS (SELECT * FROM ventas WHERE total > 1000) SELECT * FROM ventas_altas;` → CTE básica (WITH)
- `WITH RECURSIVE jerarquia AS (SELECT id, nombre, 1 AS nivel FROM empleados WHERE jefe_id IS NULL UNION ALL SELECT e.id, e.nombre, j.nivel + 1 FROM empleados e JOIN jerarquia j ON e.jefe_id = j.id) SELECT * FROM jerarquia;` → CTE recursiva (jerarquías, árboles)

> **IN vs EXISTS:** `EXISTS` suele ser más rápido con subconsultas grandes porque puede usar "semi join" y detenerse en la primera coincidencia. `IN` con valores constantes es eficiente. `NOT IN` se comporta mal si hay NULLs — prefiere `NOT EXISTS`.

## 10. Window Functions

- `SELECT col, ROW_NUMBER() OVER (ORDER BY fecha) AS rn FROM tabla;` → Numerar filas secuencialmente
- `SELECT col, RANK() OVER (ORDER BY salario DESC) AS puesto FROM empleados;` → Ranking con empates (deja huecos)
- `SELECT col, DENSE_RANK() OVER (ORDER BY salario DESC) AS puesto FROM empleados;` → Ranking sin huecos
- `SELECT col, NTILE(4) OVER (ORDER BY total) AS cuartil FROM ventas;` → Dividir en N grupos (cuartiles, percentiles)
- `SELECT col, LAG(col, 1) OVER (ORDER BY fecha) AS anterior FROM ventas;` → Fila anterior (comparar con periodo previo)
- `SELECT col, LEAD(col, 1) OVER (ORDER BY fecha) AS siguiente FROM ventas;` → Fila siguiente
- `SELECT col, SUM(col) OVER (PARTITION BY categoria) AS total_cat FROM ventas;` → Agregado por grupo sin colapsar filas
- `SELECT col, SUM(col) OVER (ORDER BY fecha ROWS BETWEEN UNBOUNDED PRECEDING AND CURRENT ROW) AS acumulado FROM ventas;` → Acumulado (running total)

> **Window Functions vs GROUP BY:** Las funciones de ventana NO colapsan filas — cada fila conserva su identidad y se le agrega el resultado del cálculo sobre su "ventana". `PARTITION BY` define el grupo (como GROUP BY pero sin colapsar). `ORDER BY` dentro del OVER define el orden dentro de la ventana.

## 11. Buenas Prácticas y Diagnóstico

- `EXPLAIN ANALYZE SELECT ...;` → Ver plan de ejecución real (dónde se gasta tiempo)
- `EXPLAIN (ANALYZE, BUFFERS) SELECT ...;` → Plan detallado con caché (PostgreSQL)
- `SELECT * FROM pg_stat_activity;` → Conexiones activas (PostgreSQL) / `SHOW PROCESSLIST` en MySQL
- `SELECT relname, seq_scan, idx_scan FROM pg_stat_user_tables;` → Tablas sin índices usados (PostgreSQL)
- `SELECT * FROM information_schema.tables WHERE table_schema = 'public';` → Metadatos estándar ANSI

> **Buenas prácticas:** Preferir `EXISTS` sobre `IN` con subconsultas. Siempre listar columnas en el INSERT (nunca `INSERT INTO VALUES (...)` sin columnas). Usar `VACUUM` en PostgreSQL periódicamente. `SELECT *` solo para exploración — en producción siempre especificar columnas. Los índices aceleran lecturas pero ralentizan escrituras. Hacer `EXPLAIN ANALYZE` antes de optimizar.

> **Cada motor tiene dialecto:** PostgreSQL tiene `RETURNING`, `ON CONFLICT`, `ILIKE`, `ARRAY_AGG`, `STRING_AGG`. MySQL tiene `ON DUPLICATE KEY`, `GROUP_CONCAT`. SQL Server usa `TOP` en vez de `LIMIT`. Siempre consultar la documentación del motor específico.
