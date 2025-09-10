---
Fecha de creación: 2025-09-05T18:34:00
Materia:
  - Base de Datos I (Relacionales)
Fecha de clase: 2025-09-05
---
# Funciones de Control de Flujo y Condición

## ``CASE``

Evalúa una condición y dependiendo del resultado nos devuelve un valor u otro.

**Sintaxis:**

```sql
CASE valor WHEN [valor_a_comparar] THEN resultado [WHEN [valor_a_comparar] THEN resultado ...] [ELSE resultado] END;
```

```sql
CASE WHEN [condición] THEN resultado [WHEN [condición] THEN resultado ...] [ELSE resultado] END;
```

**Ejemplo:**

```sql
SELECT title, rating,
CASE rating WHEN 'G' THEN 'Para todos los públicos'
WHEN 'PG' THEN 'Menores acompañados'
WHEN 'R' THEN 'Para adultos'
ELSE 'Otros'
END tipo
FROM film;
```

## ``IF``

Si expresión 1 es cierto devuelve ``valor_si_cierto`` y si no devuelve ``valor_si_falso``.

**Sintaxis:**

```sql
IF(expr1,valor_si_cierto,valor_si_falso);
```

**Ejemplo:**

```sql
SELECT title,
IF(replacement_cost > 20, CONCAT('Descuento: ', replacement_cost * 0.9), replacement_cost) coste
FROM film;
```

```sql
SELECT
IFNULL(8,0), IFNULL(NULL, 0), 0+ NULL, CONCAT('Ana', NULL);
```

**Ejemplo combinado:**

```sql
SELECT *,
CASE WHEN amount < 4 THEN 'bajo' WHEN amount < 7 THEN 'medio' ELSE 'alto' END AS tipo,
IF (amount < 6, 'Barato', 'Caro') precio,
IF (amount BETWEEN 4 AND 6, 'Medio', '') precio2,
IF (amount < 4, 'Bajo', IF(amount < 7, 'Medio', 'Alto')) tipo2
FROM sakila.payment;
```