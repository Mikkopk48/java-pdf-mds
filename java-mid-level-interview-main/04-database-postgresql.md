## 🐘 Bases de Datos y PostgreSQL

[⬆️ Volver al índice](./README.md)

---

## 📑 Contenidos de esta sección

### 🔹 SQL General
1. [Tipos de JOIN](#1-cuáles-son-los-tipos-de-join-y-cuándo-usar-cada-uno)
2. [Normalización](#2-qué-es-la-normalización-de-bases-de-datos)
3. [Window Functions](#3-qué-son-las-window-functions)
4. [Índices](#4-qué-son-los-índices-y-cuándo-usarlos)
5. [ACID](#5-qué-es-acid-y-por-qué-es-importante)

### 🔹 PostgreSQL Específico
6. [EXPLAIN](#6-qué-es-explain-y-explain-analyze)
7. [TEXT vs VARCHAR vs CHAR](#7-cuál-es-la-diferencia-entre-text-varcharn-y-charn)
8. [JSONB](#8-qué-es-jsonb-y-cuándo-usarlo)

### 🔹 Conceptos Avanzados
9. [CTE y Recursive Queries](#9-qué-son-las-cte-common-table-expressions)
10. [Particionamiento de tablas](#10-particionamiento-de-tablas-en-postgresql)
11. [Full Text Search](#11-full-text-search-en-postgresql)
12. [Locks y Concurrencia](#12-tipos-de-locks-en-postgresql)
13. [Materialized Views](#13-qué-son-las-materialized-views)
14. [Replicación y HA](#14-replicación-y-alta-disponibilidad)

---

### 🔹 SQL General

#### 1. ¿Cuáles son los tipos de JOIN y cuándo usar cada uno?

**Respuesta:**

Los **JOINs** combinan filas de dos o más tablas basándose en una condición relacionada.

**Tipos de JOIN:**

```sql
-- Tablas de ejemplo:
-- users: id, nombre
-- orders: id, user_id, total

-- 1. INNER JOIN (solo filas que coinciden en ambas tablas)
SELECT u.nombre, o.total
FROM users u
INNER JOIN orders o ON u.id = o.user_id;
-- Devuelve solo usuarios QUE TIENEN pedidos

-- 2. LEFT JOIN (todo de la izquierda + coincidencias de la derecha)
SELECT u.nombre, o.total
FROM users u
LEFT JOIN orders o ON u.id = o.user_id;
-- Devuelve TODOS los usuarios, con pedidos si existen (NULL si no)

-- 3. RIGHT JOIN (todo de la derecha + coincidencias de la izquierda)
SELECT u.nombre, o.total
FROM users u
RIGHT JOIN orders o ON u.id = o.user_id;
-- Devuelve TODOS los pedidos, con usuario si existe

-- 4. FULL OUTER JOIN (todo de ambas tablas)
SELECT u.nombre, o.total
FROM users u
FULL OUTER JOIN orders o ON u.id = o.user_id;
-- Devuelve TODO: usuarios sin pedidos Y pedidos sin usuario

-- 5. CROSS JOIN (producto cartesiano)
SELECT u.nombre, p.nombre
FROM users u
CROSS JOIN products p;
-- Cada usuario con cada producto (10 users × 100 products = 1000 filas)
```

**Diagrama visual:**

```
INNER JOIN:     A ∩ B    (solo intersección)
LEFT JOIN:      A ∪ (A ∩ B)    (todo A + intersección)
RIGHT JOIN:     B ∪ (A ∩ B)    (todo B + intersección)
FULL JOIN:      A ∪ B    (todo de ambos)
```

**Ejemplos prácticos:**

```sql
-- Usuarios SIN pedidos (LEFT JOIN + WHERE NULL)
SELECT u.id, u.nombre
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
WHERE o.id IS NULL;

-- Usuarios CON al menos un pedido
SELECT DISTINCT u.id, u.nombre
FROM users u
INNER JOIN orders o ON u.id = o.user_id;

-- Total de pedidos por usuario (incluyendo 0)
SELECT u.nombre, COUNT(o.id) as total_pedidos
FROM users u
LEFT JOIN orders o ON u.id = o.user_id
GROUP BY u.id, u.nombre;
```

**Cuándo usar cada uno:**

| JOIN | Cuándo usar |
|------|-------------|
| **INNER** | Solo registros relacionados (usuarios CON pedidos) |
| **LEFT** | Todos los registros principales + relacionados opcionales (usuarios con/sin pedidos) |
| **RIGHT** | Raramente usado (puedes invertir LEFT JOIN) |
| **FULL** | Análisis de datos completo (poco común) |
| **CROSS** | Combinaciones de productos, reportes matriciales |

---

#### 2. ¿Qué es la Normalización de bases de datos?

**Respuesta:**

La **Normalización** es el proceso de organizar las tablas y columnas de una base de datos para **reducir redundancia** y **mejorar la integridad de los datos**. Se aplica mediante **Formas Normales** (NF).

**Objetivos:**
- Eliminar duplicación de datos.
- Asegurar dependencias lógicas entre datos.
- Facilitar mantenimiento y actualizaciones.

**Formas Normales principales:**

**1NF (Primera Forma Normal) - Atomicidad**

Cada columna debe contener **valores atómicos** (indivisibles), sin listas o arrays.

```sql
-- ❌ NO cumple 1NF (columna con múltiples valores)
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    phones VARCHAR(255)  -- "555-1234, 555-5678, 555-9999" ❌
);

-- ✅ Cumple 1NF
CREATE TABLE users (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100)
);

CREATE TABLE user_phones (
    id SERIAL PRIMARY KEY,
    user_id INTEGER REFERENCES users(id),
    phone VARCHAR(20)  -- Un teléfono por fila ✅
);
```

**2NF (Segunda Forma Normal) - Sin dependencias parciales**

Debe cumplir **1NF** + Todas las columnas no-clave deben depender de **toda la clave primaria**, no solo de parte de ella.

```sql
-- ❌ NO cumple 2NF (clave compuesta: order_id + product_id)
CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER,
    product_name VARCHAR(100),  -- ❌ Depende solo de product_id
    product_price DECIMAL,      -- ❌ Depende solo de product_id
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
-- Problema: product_name se duplica si el mismo producto está en múltiples órdenes

-- ✅ Cumple 2NF
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    price DECIMAL
);

CREATE TABLE order_items (
    order_id INTEGER,
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    PRIMARY KEY (order_id, product_id)
);
```

**3NF (Tercera Forma Normal) - Sin dependencias transitivas**

Debe cumplir **2NF** + Las columnas no-clave NO deben depender de otras columnas no-clave.

```sql
-- ❌ NO cumple 3NF
CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER,
    customer_name VARCHAR(100),    -- ❌ Depende de customer_id (no-clave)
    customer_email VARCHAR(100),   -- ❌ Depende de customer_id
    total DECIMAL,
    created_at TIMESTAMP
);
-- Problema: Si cambio el nombre del cliente, debo actualizarlo en TODAS sus órdenes

-- ✅ Cumple 3NF
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100)
);

CREATE TABLE orders (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    total DECIMAL,
    created_at TIMESTAMP
);
```

**Ejemplo completo:**

```sql
-- ❌ Tabla desnormalizada (redundancia masiva)
CREATE TABLE sales_flat (
    sale_id INTEGER,
    sale_date DATE,
    customer_name VARCHAR(100),
    customer_email VARCHAR(100),
    customer_city VARCHAR(100),
    product_name VARCHAR(100),
    product_category VARCHAR(100),
    quantity INTEGER,
    unit_price DECIMAL
);
-- Problemas:
-- - customer_name, email, city se repiten en cada venta
-- - product_name, category se repiten en cada venta
-- - Si cambio el email del cliente, debo actualizar 100 filas

-- ✅ Modelo normalizado (3NF)
CREATE TABLE customers (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    email VARCHAR(100),
    city VARCHAR(100)
);

CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name VARCHAR(100),
    category VARCHAR(100)
);

CREATE TABLE sales (
    id SERIAL PRIMARY KEY,
    customer_id INTEGER REFERENCES customers(id),
    sale_date DATE
);

CREATE TABLE sale_items (
    id SERIAL PRIMARY KEY,
    sale_id INTEGER REFERENCES sales(id),
    product_id INTEGER REFERENCES products(id),
    quantity INTEGER,
    unit_price DECIMAL  -- Precio al momento de la venta
);
```

**¿Cuándo NO normalizar (Desnormalización)?**

En sistemas de **lectura intensiva** (reporting, analytics), a veces conviene **desnormalizar** para mejorar performance:

```sql
-- Tabla desnormalizada para reportes (evita JOINs)
CREATE MATERIALIZED VIEW sales_report AS
SELECT 
    s.id,
    s.sale_date,
    c.name AS customer_name,
    c.city AS customer_city,
    p.name AS product_name,
    p.category AS product_category,
    si.quantity,
    si.unit_price
FROM sales s
JOIN customers c ON s.customer_id = c.id
JOIN sale_items si ON s.id = si.sale_id
JOIN products p ON si.product_id = p.id;

-- Refrescar periódicamente
REFRESH MATERIALIZED VIEW sales_report;
```

**Resumen:**

| Forma | Regla | Problema que resuelve |
|-------|-------|----------------------|
| **1NF** | Valores atómicos | Columnas con listas |
| **2NF** | 1NF + Sin dependencias parciales | Redundancia por claves compuestas |
| **3NF** | 2NF + Sin dependencias transitivas | Redundancia entre columnas no-clave |

---

#### 3. ¿Qué son las Window Functions?

**Respuesta:**

Las **Window Functions** (funciones de ventana) permiten realizar cálculos **a través de un conjunto de filas** relacionadas con la fila actual, sin agruparlas (a diferencia de `GROUP BY`).

**Sintaxis básica:**
```sql
FUNCTION() OVER (
    PARTITION BY column  -- Divide en grupos (opcional)
    ORDER BY column      -- Orden dentro de cada grupo (opcional)
    ROWS/RANGE ...       -- Define el "marco de ventana" (opcional)
)
```

**Funciones más comunes:**

**1. ROW_NUMBER() - Número de fila**

Asigna un número único a cada fila dentro de una partición.

```sql
SELECT 
    name,
    department,
    salary,
    ROW_NUMBER() OVER (
        PARTITION BY department 
        ORDER BY salary DESC
    ) AS row_num
FROM employees;

/* Resultado:
name       | department | salary | row_num
-----------|------------|--------|--------
Juan       | IT         | 80000  | 1
María      | IT         | 75000  | 2
Pedro      | IT         | 70000  | 3
Ana        | Sales      | 65000  | 1
Luis       | Sales      | 60000  | 2
*/

-- Obtener el empleado mejor pagado de cada departamento:
WITH ranked AS (
    SELECT *, ROW_NUMBER() OVER (PARTITION BY department ORDER BY salary DESC) AS rn
    FROM employees
)
SELECT name, department, salary
FROM ranked
WHERE rn = 1;
```

**2. RANK() y DENSE_RANK() - Ranking con empates**

```sql
SELECT 
    name,
    score,
    RANK() OVER (ORDER BY score DESC) AS rank,
    DENSE_RANK() OVER (ORDER BY score DESC) AS dense_rank
FROM students;

/* Diferencia:
score | RANK | DENSE_RANK
------|------|------------
100   | 1    | 1
100   | 1    | 1  (mismo score)
95    | 3    | 2  ← RANK salta, DENSE_RANK no
90    | 4    | 3
*/
```

**3. LAG() y LEAD() - Acceder a filas anteriores/siguientes**

```sql
-- Comparar ventas con el mes anterior
SELECT 
    month,
    sales,
    LAG(sales) OVER (ORDER BY month) AS prev_month_sales,
    sales - LAG(sales) OVER (ORDER BY month) AS difference
FROM monthly_sales;

/* Resultado:
month   | sales | prev_month_sales | difference
--------|-------|------------------|------------
2024-01 | 10000 | NULL             | NULL
2024-02 | 12000 | 10000            | 2000
2024-03 | 11000 | 12000            | -1000
*/
```

**4. SUM(), AVG(), COUNT() - Agregaciones acumulativas**

```sql
-- Suma acumulada (running total)
SELECT 
    date,
    amount,
    SUM(amount) OVER (ORDER BY date) AS running_total
FROM transactions;

/* Resultado:
date       | amount | running_total
-----------|--------|---------------
2024-01-01 | 100    | 100
2024-01-02 | 150    | 250
2024-01-03 | 200    | 450
*/

-- Promedio móvil (últimas 3 filas)
SELECT 
    date,
    price,
    AVG(price) OVER (
        ORDER BY date 
        ROWS BETWEEN 2 PRECEDING AND CURRENT ROW
    ) AS moving_avg_3
FROM stock_prices;
```

**5. FIRST_VALUE() y LAST_VALUE() - Primer/último valor**

```sql
-- Comparar cada venta con la primera del año
SELECT 
    month,
    sales,
    FIRST_VALUE(sales) OVER (ORDER BY month) AS first_month,
    sales - FIRST_VALUE(sales) OVER (ORDER BY month) AS diff_from_first
FROM monthly_sales
WHERE EXTRACT(YEAR FROM month) = 2024;
```

**Caso de uso real: Top N por categoría**

```sql
-- Los 3 productos más vendidos por categoría
WITH ranked_products AS (
    SELECT 
        product_name,
        category,
        total_sales,
        ROW_NUMBER() OVER (
            PARTITION BY category 
            ORDER BY total_sales DESC
        ) AS rank_in_category
    FROM product_sales
)
SELECT product_name, category, total_sales
FROM ranked_products
WHERE rank_in_category <= 3;
```

**Ventajas sobre GROUP BY:**

```sql
-- ❌ GROUP BY: Pierdes el detalle de cada fila
SELECT department, AVG(salary) AS avg_salary
FROM employees
GROUP BY department;

/* Resultado:
department | avg_salary
-----------|------------
IT         | 75000
Sales      | 62500
*/

-- ✅ Window Function: Mantienes el detalle + añades agregación
SELECT 
    name,
    department,
    salary,
    AVG(salary) OVER (PARTITION BY department) AS dept_avg_salary,
    salary - AVG(salary) OVER (PARTITION BY department) AS diff_from_avg
FROM employees;

/* Resultado:
name  | department | salary | dept_avg_salary | diff_from_avg
------|------------|--------|-----------------|---------------
Juan  | IT         | 80000  | 75000           | 5000
María | IT         | 75000  | 75000           | 0
Pedro | IT         | 70000  | 75000           | -5000
Ana   | Sales      | 65000  | 62500           | 2500
*/
```

**Best Practices:**
- ✅ Usa `ROW_NUMBER()` para paginación custom o eliminar duplicados.
- ✅ Usa `LAG/LEAD` para comparaciones temporales (mes vs mes anterior).
- ✅ Usa window functions en lugar de subqueries complejas (más legible).
- ⚠️ Cuidado con performance en tablas muy grandes sin índices en `ORDER BY`.

---

#### 4. ¿Qué son los índices y cuándo usarlos?

**Respuesta:**

Un **índice** es una estructura de datos (generalmente **B-Tree**) que mejora la velocidad de las operaciones de **lectura** (SELECT) a costa de ralentizar las **escrituras** (INSERT/UPDATE/DELETE) y ocupar espacio adicional en disco.

**Analogía:** Es como el índice de un libro. En lugar de leer todo el libro para encontrar un tema, consultas el índice que te dice exactamente en qué página está.

**¿Cómo funciona?**

Sin índice (Sequential Scan):
```sql
SELECT * FROM users WHERE email = 'juan@mail.com';
-- Recorre TODAS las filas (1 millón de filas = lento)
-- Complejidad: O(n)
```

Con índice (Index Scan):
```sql
CREATE INDEX idx_users_email ON users(email);

SELECT * FROM users WHERE email = 'juan@mail.com';
-- Va directo a la fila usando el índice (árbol binario)
-- Complejidad: O(log n)
```

**Tipos de índices en PostgreSQL:**

**1. B-Tree (default, más común):**
```sql
CREATE INDEX idx_users_email ON users(email);
-- Ideal para: =, <, <=, >, >=, BETWEEN, IN, ORDER BY
```

**2. Hash (solo igualdad):**
```sql
CREATE INDEX idx_users_email ON users USING HASH(email);
-- Solo para operador =
-- Más rápido que B-Tree para igualdad exacta, pero menos flexible
```

**3. GIN (Generalized Inverted Index - JSONB, arrays, full-text):**
```sql
CREATE INDEX idx_users_data ON users USING GIN(data);
-- Para JSONB, arrays, búsqueda de texto completo
```

**4. GiST (Generalized Search Tree - datos geométricos, full-text):**
```sql
CREATE INDEX idx_locations ON stores USING GIST(location);
-- Para datos geoespaciales, rangos
```

**5. Índice compuesto (múltiples columnas):**
```sql
CREATE INDEX idx_users_last_first ON users(last_name, first_name);
-- Útil para búsquedas por apellido Y nombre
```

**6. Índice parcial (con condición WHERE):**
```sql
CREATE INDEX idx_active_users ON users(email) WHERE active = true;
-- Solo indexa usuarios activos (ahorra espacio)
```

**7. Índice único:**
```sql
CREATE UNIQUE INDEX idx_users_email ON users(email);
-- Garantiza valores únicos (equivalente a constraint UNIQUE)
```

**¿Cuándo crear índices?**

✅ **SÍ crear índice:**
- Columnas en **WHERE** frecuentes (`WHERE email = ...`)
- Columnas en **JOIN** (`ON u.id = o.user_id`)
- Columnas en **ORDER BY** (`ORDER BY created_at DESC`)
- **Foreign Keys** (siempre)
- Columnas con **alta cardinalidad** (muchos valores únicos: email, DNI, ID)

❌ **NO crear índice:**
- Tablas pequeñas (< 1000 filas)
- Columnas con **baja cardinalidad** (pocos valores: género con [M, F, O])
- Columnas que casi nunca se consultan
- Tablas con muchas escrituras y pocas lecturas

**Impacto:**

```sql
-- Sin índice en tabla de 1 millón de filas
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@mail.com';
-- Seq Scan on users  (cost=0.00..25000.00 rows=1 width=100) (actual time=250.123..250.125 rows=1)
-- Planning Time: 0.100 ms
-- Execution Time: 250.300 ms  ❌ MUY LENTO

-- Con índice
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@mail.com';
-- Index Scan using idx_users_email on users  (cost=0.42..8.44 rows=1 width=100) (actual time=0.015..0.017 rows=1)
-- Planning Time: 0.100 ms
-- Execution Time: 0.050 ms  ✅ 5000x MÁS RÁPIDO
```

**Best Practices:**
- ✅ Crea índices basándote en queries reales (no por adivinar).
- ✅ Usa `EXPLAIN ANALYZE` para verificar si se usan.
- ✅ Monitorea índices no usados: ocupan espacio y ralentizan escrituras.
- ❌ No crees índices en todas las columnas (overhead de escritura).

```sql
-- Ver índices no usados
SELECT schemaname, tablename, indexname
FROM pg_stat_user_indexes
WHERE idx_scan = 0 AND indexrelname NOT LIKE 'pg_toast%';
```

**Índices Compuestos (Composite Indexes):**

Un **índice compuesto** incluye **múltiples columnas**. Son útiles cuando filtras o ordenas por varias columnas simultáneamente.

```sql
-- Índice compuesto en (last_name, first_name)
CREATE INDEX idx_users_name ON users(last_name, first_name);

-- Índice compuesto en (category, price)
CREATE INDEX idx_products_category_price ON products(category, price);
```

**Regla crítica: Leftmost Prefix Rule (Prefijo más a la izquierda)**

El índice compuesto solo se usa si la query incluye **las columnas más a la izquierda** del índice.

```sql
CREATE INDEX idx_users_city_age_status ON users(city, age, status);

-- ✅ Usa el índice (empieza por 'city')
SELECT * FROM users WHERE city = 'Madrid';
SELECT * FROM users WHERE city = 'Madrid' AND age > 25;
SELECT * FROM users WHERE city = 'Madrid' AND age > 25 AND status = 'active';

-- ✅ Usa el índice parcialmente (city + age)
SELECT * FROM users WHERE city = 'Madrid' AND age > 25;

-- ❌ NO usa el índice (no incluye 'city', la primera columna)
SELECT * FROM users WHERE age > 25;
SELECT * FROM users WHERE status = 'active';
SELECT * FROM users WHERE age > 25 AND status = 'active';
```

**Orden de columnas importante:**

Coloca primero las columnas más **selectivas** (que filtran más filas).

```sql
-- ❌ MAL: gender tiene baja cardinalidad (M/F/O)
CREATE INDEX idx_bad ON users(gender, city, age);

-- ✅ BIEN: city es más selectivo que gender
CREATE INDEX idx_good ON users(city, age, gender);
```

**Guía de orden:**
1. **Igualdad (=)** primero.
2. **Rango (>, <, BETWEEN)** después.
3. **Columnas más selectivas** primero.

```sql
-- Query:
SELECT * FROM orders 
WHERE customer_id = 123 
  AND status = 'pending' 
  AND created_at > '2024-01-01';

-- ✅ Índice óptimo:
CREATE INDEX idx_orders_optimal ON orders(customer_id, status, created_at);
-- Razón: customer_id y status son igualdad (=), created_at es rango (>)
```

**Casos de uso:**

```sql
-- Caso 1: Búsquedas frecuentes por categoría + precio
CREATE INDEX idx_products_cat_price ON products(category, price DESC);

SELECT * FROM products 
WHERE category = 'Electronics' 
ORDER BY price DESC 
LIMIT 10;  -- Usa el índice para filtrar Y ordenar

-- Caso 2: Queries con múltiples filtros
CREATE INDEX idx_users_signup ON users(country, signup_date, status);

SELECT * FROM users 
WHERE country = 'ES' 
  AND signup_date > '2024-01-01' 
  AND status = 'active';
```

**¿Cuándo NO usar índices compuestos?**

- Columnas poco correlacionadas en queries.
- Demasiadas combinaciones diferentes de filtros (crea índices simples en su lugar).

**Verificar uso:**
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE city = 'Madrid' AND age > 25;
-- Busca "Index Scan using idx_users_city_age_status"
```

---

#### 5. ¿Qué es ACID y por qué es importante?

**Respuesta:**

**ACID** son las cuatro propiedades que garantizan que las transacciones en una base de datos sean **fiables** y **consistentes**, incluso ante fallos del sistema.

**A - Atomicity (Atomicidad):**
- **Todo o nada**: Una transacción se completa completamente o no se aplica en absoluto.
- Si cualquier parte falla → toda la transacción se revierte (ROLLBACK).

```sql
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
UPDATE accounts SET balance = balance + 100 WHERE id = 2;
-- Si la segunda falla (ej: cuenta 2 no existe) → se revierte la primera también
COMMIT;
```

**C - Consistency (Consistencia):**
- La BD pasa de un **estado válido** a otro **estado válido**.
- Se respetan todas las reglas: constraints, triggers, cascades, foreign keys.

```sql
-- Constraint de saldo mínimo
ALTER TABLE accounts ADD CONSTRAINT check_balance CHECK (balance >= 0);

BEGIN;
UPDATE accounts SET balance = balance - 500 WHERE id = 1;
-- Si balance resultante < 0 → ERROR, ROLLBACK
COMMIT;
```

**I - Isolation (Aislamiento):**
- Las transacciones **concurrentes** no se interfieren entre sí.
- Cada transacción se ejecuta como si fuera la única en el sistema.
- Depende del **nivel de aislamiento** configurado.

```sql
-- Transacción 1
BEGIN;
UPDATE accounts SET balance = balance - 100 WHERE id = 1;
-- Transacción 2 NO ve este cambio hasta el COMMIT
COMMIT;
```

**D - Durability (Durabilidad):**
- Una vez que se hace **COMMIT**, los cambios persisten **permanentemente**.
- Sobreviven a caídas del sistema, cortes de luz, etc.
- Se guardan en disco (no solo en RAM).

```sql
BEGIN;
INSERT INTO logs (message) VALUES ('Operación crítica completada');
COMMIT;
-- Aunque el servidor se caiga 1 segundo después, el registro persiste ✅
```

**Niveles de Aislamiento (Isolation Levels):**

| Nivel | Dirty Read | Non-Repeatable Read | Phantom Read | Concurrencia | Consistencia |
|-------|------------|---------------------|--------------|--------------|--------------|
| **READ UNCOMMITTED** | ✅ Sí | ✅ Sí | ✅ Sí | Alta | Baja |
| **READ COMMITTED** | ❌ No | ✅ Sí | ✅ Sí | Media-Alta | Media |
| **REPEATABLE READ** | ❌ No | ❌ No | ✅ Sí | Media | Alta |
| **SERIALIZABLE** | ❌ No | ❌ No | ❌ No | Baja | Máxima |

**Problemas de concurrencia:**

**1. Dirty Read (lectura sucia):**
Leer datos de una transacción no confirmada.
```sql
-- T1: BEGIN;
-- T1: UPDATE accounts SET balance = 1000 WHERE id = 1;
-- T2: SELECT balance FROM accounts WHERE id = 1;  -- Lee 1000 (NO CONFIRMADO)
-- T1: ROLLBACK;  -- T2 leyó datos que nunca existieron ❌
```

**2. Non-Repeatable Read:**
Leer la misma fila dos veces y obtener valores diferentes.
```sql
-- T1: SELECT balance FROM accounts WHERE id = 1;  -- Resultado: 500
-- T2: UPDATE accounts SET balance = 1000 WHERE id = 1; COMMIT;
-- T1: SELECT balance FROM accounts WHERE id = 1;  -- Resultado: 1000 (cambió!)
```

**3. Phantom Read:**
Una query devuelve diferentes filas en la misma transacción.
```sql
-- T1: SELECT COUNT(*) FROM users WHERE age > 18;  -- Resultado: 100
-- T2: INSERT INTO users (age) VALUES (25); COMMIT;
-- T1: SELECT COUNT(*) FROM users WHERE age > 18;  -- Resultado: 101 (nueva fila)
```

**PostgreSQL default: READ COMMITTED**

```sql
-- Cambiar nivel de aislamiento
BEGIN TRANSACTION ISOLATION LEVEL SERIALIZABLE;
-- Tu código
COMMIT;
```

---

#### 6. ¿Qué es EXPLAIN y EXPLAIN ANALYZE?

**Respuesta:**

`EXPLAIN` y `EXPLAIN ANALYZE` son comandos **fundamentales** para optimizar queries. Muestran cómo PostgreSQL ejecuta una consulta.

**EXPLAIN (plan estimado, NO ejecuta la query):**

```sql
EXPLAIN SELECT * FROM users WHERE email = 'test@mail.com';

-- Resultado:
-- Seq Scan on users  (cost=0.00..35.50 rows=1 width=100)
--   Filter: (email = 'test@mail.com'::text)
```

**EXPLAIN ANALYZE (plan real, SÍ ejecuta la query):**

```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE email = 'test@mail.com';

-- Resultado:
-- Seq Scan on users  (cost=0.00..35.50 rows=1 width=100) (actual time=0.015..2.456 rows=1 loops=1)
--   Filter: (email = 'test@mail.com'::text)
--   Rows Removed by Filter: 999
-- Planning Time: 0.082 ms
-- Execution Time: 2.478 ms
```

**Componentes del plan:**

**1. Tipo de escaneo:**
- **Seq Scan**: Escaneo secuencial (lee toda la tabla) ❌ Lento para tablas grandes
- **Index Scan**: Usa índice ✅ Rápido
- **Index Only Scan**: Usa solo el índice, no toca la tabla ✅ Muy rápido
- **Bitmap Index Scan + Bitmap Heap Scan**: Para múltiples índices
- **Nested Loop**: JOIN anidado (tabla pequeña)
- **Hash Join**: JOIN con tabla hash (tablas medianas)
- **Merge Join**: JOIN ordenado (tablas grandes)

**2. cost (costo estimado):**
```
cost=0.00..35.50
     ^^^    ^^^
     inicio  final
```
- Unidades arbitrarias (no son milisegundos).
- Comparables entre planes de la misma query.
- Menor = mejor.

**3. rows (filas estimadas):**
- Cuántas filas espera devolver.
- Si está muy alejado de `actual rows` → Estadísticas desactualizadas.

**4. width (ancho promedio de fila en bytes):**
- Tamaño estimado de cada fila.

**5. actual time (tiempo real, solo con ANALYZE):**
```
actual time=0.015..2.456
            ^^^    ^^^
            primera fila  última fila
```

**6. loops:**
- Cuántas veces se ejecutó este nodo.
- loops > 1 en Nested Loop puede ser problemático.

**Ejemplos prácticos:**

**Problema: Seq Scan en tabla grande**
```sql
EXPLAIN ANALYZE SELECT * FROM orders WHERE customer_id = 123;

-- Seq Scan on orders  (cost=0.00..50000.00 rows=10 width=200) (actual time=500.123..500.456 rows=10 loops=1)
--   Filter: (customer_id = 123)
--   Rows Removed by Filter: 999990

-- Solución: Crear índice
CREATE INDEX idx_orders_customer_id ON orders(customer_id);

-- Nuevo plan:
-- Index Scan using idx_orders_customer_id on orders  (cost=0.42..8.44 rows=10 width=200) (actual time=0.015..0.025 rows=10 loops=1)
```

**Problema: Rows estimado ≠ Rows actual**
```sql
EXPLAIN ANALYZE SELECT * FROM users WHERE status = 'active';

-- Seq Scan on users  (cost=0.00..35.50 rows=500 width=100) (actual time=0.015..2.456 rows=50000 loops=1)
--                                      ^^^estimado            ^^^real (100x más!)

-- Solución: Actualizar estadísticas
ANALYZE users;
```

**Opciones adicionales:**

```sql
-- Más detalles
EXPLAIN (ANALYZE true, BUFFERS true, VERBOSE true) SELECT ...;

-- Formato JSON
EXPLAIN (FORMAT JSON) SELECT ...;
```

**Best Practices:**
- ✅ Usa EXPLAIN ANALYZE en desarrollo para optimizar queries lentas.
- ✅ Busca Seq Scan en tablas grandes → Considera índices.
- ✅ Verifica que `rows` estimado ≈ `actual rows`.
- ✅ Ejecuta `ANALYZE` periódicamente para actualizar estadísticas.

---

### 🔹 PostgreSQL Específico

#### 7. ¿Cuál es la diferencia entre TEXT, VARCHAR(n) y CHAR(n)?

**Respuesta:**

| Tipo | Límite | Padding | Storage | Rendimiento | Uso |
|------|--------|---------|---------|-------------|-----|
| **TEXT** | Sin límite práctico (1GB) | No | Variable | Igual a VARCHAR | ✅ Recomendado |
| **VARCHAR(n)** | n caracteres | No | Variable | Igual a TEXT | Validación de longitud |
| **CHAR(n)** | Exactamente n | Sí (con espacios) | Fijo | Ligeramente más lento | Raramente útil |

**En PostgreSQL: TEXT ≈ VARCHAR**

A diferencia de otros DBs (MySQL, SQL Server), en PostgreSQL no hay diferencia de rendimiento entre TEXT y VARCHAR.

```sql
-- Todos funcionan igual
CREATE TABLE users (
    bio TEXT,                    -- ✅ Recomendado (sin límite)
    email VARCHAR(255),          -- ✅ OK (con validación de longitud)
    code CHAR(5)                 -- ❌ Raramente necesario
);

-- Internamente, PostgreSQL los trata casi igual
```

**VARCHAR(n) vs TEXT:**

```sql
-- VARCHAR con límite
ALTER TABLE users ADD COLUMN username VARCHAR(50);
INSERT INTO users (username) VALUES ('NombreMuyLargoQueExcede50Caracteres');
-- ERROR: value too long for type character varying(50)

-- TEXT sin límite
ALTER TABLE users ADD COLUMN bio TEXT;
INSERT INTO users (bio) VALUES ('Texto de cualquier longitud...');
-- ✅ OK
```

**CHAR(n) (rellena con espacios):**

```sql
CREATE TABLE codes (
    code CHAR(5)
);

INSERT INTO codes VALUES ('AB');   -- Se guarda como 'AB   ' (con 3 espacios)
INSERT INTO codes VALUES ('ABCDE'); -- Se guarda como 'ABCDE'
INSERT INTO codes VALUES ('ABCDEF'); -- ERROR: value too long

SELECT code, length(code) FROM codes;
-- 'AB   ' | 2  (length ignora espacios trailing)
-- 'ABCDE' | 5
```

**Cuándo usar cada uno:**

| Tipo | Cuándo usar |
|------|-------------|
| **TEXT** | Caso por defecto (descripciones, comentarios, JSON, etc.) |
| **VARCHAR(n)** | Cuando necesitas validación de longitud (username, email) |
| **CHAR(n)** | Códigos de longitud fija (código postal, ISO country codes) |

**Recomendación PostgreSQL:**
> "Use TEXT. VARCHAR(n) adds a check constraint but has no performance benefit."

```sql
-- ✅ BIEN: Usa TEXT + CHECK si necesitas validación
CREATE TABLE users (
    email TEXT CHECK (length(email) <= 255),
    bio TEXT
);
```

---

#### 8. ¿Qué es JSONB y cuándo usarlo?

**Respuesta:**

**JSONB** es un tipo de datos en PostgreSQL para almacenar **JSON en formato binario**. Es más eficiente que el tipo `JSON` (que guarda texto plano).

**JSON vs JSONB:**

| Característica | JSON | JSONB |
|----------------|------|-------|
| **Storage** | Texto plano | Binario (descompuesto) |
| **Indexable** | ❌ No | ✅ Sí (índices GIN) |
| **Performance** | Lento (parsea cada vez) | Rápido (pre-parseado) |
| **Orden de keys** | Preserva | No preserva |
| **Espacios** | Preserva | Elimina |
| **Duplicados** | Permite keys duplicadas | Elimina duplicados |
| **Uso** | ❌ Raramente | ✅ Siempre usar JSONB |

**Crear tabla con JSONB:**

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT,
    attributes JSONB  -- Color, tamaño, etc. (datos semi-estructurados)
);

-- Insertar
INSERT INTO products (name, attributes) VALUES
    ('Camiseta', '{"color": "rojo", "talla": "M", "precio": 19.99}'),
    ('Pantalón', '{"color": "azul", "talla": "L", "material": "algodón"}');
```

**Consultar JSONB:**

```sql
-- Operador -> devuelve JSON
SELECT attributes -> 'color' FROM products;  -- "rojo", "azul"

-- Operador ->> devuelve TEXT
SELECT attributes ->> 'color' FROM products;  -- rojo, azul

-- Buscar por valor
SELECT * FROM products WHERE attributes ->> 'color' = 'rojo';

-- Buscar clave existente
SELECT * FROM products WHERE attributes ? 'material';  -- Solo el pantalón

-- Buscar en nested JSON
SELECT * FROM products WHERE attributes -> 'specs' ->> 'weight' = '500g';

-- @> contiene
SELECT * FROM products WHERE attributes @> '{"color": "rojo"}';

-- <@ está contenido en
SELECT * FROM products WHERE '{"color": "rojo"}' <@ attributes;
```

**Modificar JSONB:**

```sql
-- Añadir/actualizar campo
UPDATE products 
SET attributes = attributes || '{"stock": 100}'
WHERE id = 1;

-- Eliminar campo
UPDATE products 
SET attributes = attributes - 'precio'
WHERE id = 1;

-- Actualizar campo anidado
UPDATE products 
SET attributes = jsonb_set(attributes, '{specs,weight}', '"600g"')
WHERE id = 1;
```

**Índices en JSONB (CLAVE para performance):**

```sql
-- 1. Índice GIN genérico (operadores @>, ?, ?&, ?|)
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- Ahora esto es rápido:
SELECT * FROM products WHERE attributes @> '{"color": "rojo"}';

-- 2. Índice en un campo específico
CREATE INDEX idx_products_color ON products ((attributes ->> 'color'));

-- Ahora esto es rápido:
SELECT * FROM products WHERE attributes ->> 'color' = 'rojo';

-- 3. Índice en expresión
CREATE INDEX idx_products_price ON products ((CAST(attributes ->> 'precio' AS NUMERIC)));
```

**Funciones útiles:**

```sql
-- jsonb_each: Expandir a key-value
SELECT * FROM jsonb_each('{"a": 1, "b": 2}'::jsonb);
--  key | value
-- -----+-------
--  a   | 1
--  b   | 2

-- jsonb_array_elements: Expandir array
SELECT * FROM jsonb_array_elements('[1, 2, 3]'::jsonb);

-- jsonb_build_object: Construir JSON
SELECT jsonb_build_object('name', 'Juan', 'age', 30);
-- {"name": "Juan", "age": 30}

-- jsonb_pretty: Formatear
SELECT jsonb_pretty('{"a":1,"b":2}'::jsonb);
-- {
--     "a": 1,
--     "b": 2
-- }
```

**Cuándo usar JSONB:**

✅ **Usar JSONB cuando:**
- Datos semi-estructurados (atributos variables por producto).
- Configuraciones/preferencias de usuario.
- Logs/eventos con schema flexible.
- Integraciones con APIs externas (guardas el payload completo).
- Prototipado rápido (evitas migrations constantes).

❌ **NO usar JSONB cuando:**
- Datos altamente estructurados con relaciones (usa tablas normalizadas).
- Necesitas foreign keys, constraints complejos.
- Reportes/agregaciones frecuentes en esos campos.

**Ejemplo práctico: E-commerce**

```sql
CREATE TABLE products (
    id SERIAL PRIMARY KEY,
    name TEXT NOT NULL,
    category TEXT NOT NULL,
    base_price NUMERIC(10, 2) NOT NULL,
    
    -- Atributos variables según categoría
    attributes JSONB DEFAULT '{}'::jsonb
    -- Ropa: {"color", "talla", "material"}
    -- Electrónica: {"marca", "modelo", "garantia_meses"}
    -- Libros: {"autor", "isbn", "paginas"}
);

-- Índice para búsquedas
CREATE INDEX idx_products_attributes ON products USING GIN (attributes);

-- Búsqueda por atributo
SELECT * FROM products 
WHERE category = 'ropa' 
  AND attributes ->> 'color' = 'rojo'
  AND attributes ->> 'talla' = 'M';
```

[⬆️ Volver al índice de esta sección](#-contenidos-de-esta-sección)

---

[⬅️ Anterior: JPA y Hibernate](./03-jpa-hibernate.md) | [🏠 Volver al Inicio](./README.md) | [Siguiente: Arquitectura y Testing ➡️](./05-architecture-ops.md)