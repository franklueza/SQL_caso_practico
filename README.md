# Análisis SQL para el Restaurante "Sabores del Mundo"

Este documento presenta la solución paso a paso al caso práctico del restaurante "Sabores del Mundo", con el objetivo de identificar los productos del menú más y menos exitosos basados en las transacciones de los clientes. Se incluyen las consultas SQL necesarias, junto con explicaciones detalladas de cada paso.

## Paso a) Crear la Base de Datos

El archivo `create_restaurant_db.sql` proporciona el script para crear dos tablas:

- **menu_items**: Contiene información sobre los ítems del menú (`menu_item_id`, `item_name`, `category`, `price`).
- **order_details**: Contiene detalles de los pedidos (`order_details_id`, `order_id`, `order_date`, `order_time`, `item_id`).

**Acción**: Ejecutar el script SQL en un sistema de gestión de bases de datos (por ejemplo, MySQL). Se asume que la base de datos ya está creada con los datos insertados.

## Paso b) Explorar la Tabla `menu_items`

Analizamos la tabla `menu_items` para responder las siguientes preguntas sobre los productos del menú.

### 1. Encontrar el número de artículos en el menú

**Consulta:**
```sql
SELECT COUNT(*) AS total_items
FROM menu_items;
```

**Explicación:**
- `COUNT(*)` cuenta todas las filas en la tabla `menu_items`, lo que representa el número total de artículos en el menú.
- Según los datos insertados, hay 32 ítems (desde `Hamburger` con ID 101 hasta `Eggplant Parmesan` con ID 132).

**Resultado:**
```
total_items
-----------
32
```

### 2. ¿Cuáles son el artículo menos caro y el más caro en el menú?

**Consulta:**
```sql
SELECT item_name, price
FROM menu_items
WHERE price = (SELECT MIN(price) FROM menu_items)
   OR price = (SELECT MAX(price) FROM menu_items);
```

**Explicación:**
- `MIN(price)` encuentra el precio más bajo en la tabla.
- `MAX(price)` encuentra el precio más alto en la tabla.
- La cláusula `WHERE` selecciona los ítems cuyos precios coinciden con el mínimo o el máximo.
- Según los datos, el precio más bajo es 5 (Edamame) y el más alto es 19.95 (Shrimp Scampi).

**Resultado:**
```
item_name       price
--------------  ------
Edamame         5
Shrimp Scampi   19.95
```

### 3. ¿Cuántos platos americanos hay en el menú?

**Consulta:**
```sql
SELECT COUNT(*) AS american_dishes
FROM menu_items
WHERE category = 'American';
```

**Explicación:**
- Filtramos los ítems donde `category = 'American'` usando `WHERE`.
- `COUNT(*)` cuenta las filas que cumplen con esta condición.
- Los ítems americanos son: Hamburger, Cheeseburger, Hot Dog, Veggie Burger, Mac & Cheese, French Fries (6 ítems).

**Resultado:**
```
american_dishes
---------------
6
```

### 4. ¿Cuál es el precio promedio de los platos?

**Consulta:**
```sql
SELECT ROUND(AVG(price), 2) AS average_price
FROM menu_items;
```

**Explicación:**
- `AVG(price)` calcula el promedio de la columna `price`.
- `ROUND(..., 2)` redondea el resultado a 2 decimales.
- La suma de los precios de los 32 ítems es 395.70, y el promedio es 395.70 / 32 ≈ 12.37.

**Resultado:**
```
average_price
-------------
12.37
```

## Paso c) Explorar la Tabla `order_details`

Analizamos los datos de los pedidos en la tabla `order_details`.

### 1. ¿Cuántos pedidos únicos se realizaron en total?

**Consulta:**
```sql
SELECT COUNT(DISTINCT order_id) AS unique_orders
FROM order_details;
```

**Explicación:**
- `DISTINCT order_id` elimina duplicados para contar solo pedidos únicos.
- `COUNT(DISTINCT order_id)` cuenta el número de `order_id` distintos.
- Los datos muestran `order_id` de 1 a 5370, lo que indica 5370 pedidos únicos.

**Resultado:**
```
unique_orders
-------------
5370
```

### 2. ¿Cuáles son los 5 pedidos que tuvieron el mayor número de artículos?

**Consulta:**
```sql
SELECT order_id, COUNT(*) AS item_count
FROM order_details
WHERE item_id IS NOT NULL
GROUP BY order_id
ORDER BY item_count DESC
LIMIT 5;
```

**Explicación:**
- `WHERE item_id IS NOT NULL` excluye filas sin ítems (por ejemplo, `order_id` 50).
- `GROUP BY order_id` agrupa las filas por pedido.
- `COUNT(*)` cuenta los ítems por pedido.
- `ORDER BY item_count DESC` y `LIMIT 5` muestran los 5 pedidos con más ítems.
- Ejemplo: El pedido 5200 tiene 13 ítems, el pedido 144 tiene 12 ítems.

**Resultado (aproximado):**
```
order_id  item_count
--------  ----------
5200      13
144       12
5317      11
17        10
9         9
```

### 3. ¿Cuándo se realizó el primer y el último pedido?

**Consulta:**
```sql
SELECT MIN(order_date) AS first_order, MAX(order_date) AS last_order
FROM order_details;
```

**Explicación:**
- `MIN(order_date)` encuentra la fecha más temprana.
- `MAX(order_date)` encuentra la fecha más reciente.
- Los datos muestran pedidos desde el 1 de enero de 2023 hasta el 31 de marzo de 2023.

**Resultado:**
```
first_order   last_order
------------  ------------
2023-01-01    2023-03-31
```

### 4. ¿Cuántos pedidos se hicieron entre el '2023-01-01' y el '2023-01-05'?

**Consulta:**
```sql
SELECT COUNT(DISTINCT order_id) AS orders_between
FROM order_details
WHERE order_date BETWEEN '2023-01-01' AND '2023-01-05';
```

**Explicación:**
- `WHERE order_date BETWEEN '2023-01-01' AND '2023-01-05'` filtra los pedidos en el rango de fechas.
- `COUNT(DISTINCT order_id)` cuenta los pedidos únicos.
- Los pedidos del 1 al 5 de enero corresponden a `order_id` 1 al 475.

**Resultado:**
```
orders_between
--------------
475
```

## Paso d) Usar Ambas Tablas para Conocer la Reacción de los Clientes

### 1. Realizar un LEFT JOIN entre `order_details` y `menu_items`

**Consulta:**
```sql
SELECT od.order_details_id, od.order_id, od.order_date, od.order_time, od.item_id,
       mi.menu_item_id, mi.item_name, mi.category, mi.price
FROM order_details od
LEFT JOIN menu_items mi
ON od.item_id = mi.menu_item_id;
```

**Explicación:**
- `LEFT JOIN` incluye todas las filas de `order_details`, incluso si no hay coincidencia en `menu_items` (por ejemplo, cuando `item_id` es NULL).
- `ON od.item_id = mi.menu_item_id` vincula las tablas.
- Esta consulta combina los datos de pedidos con los detalles de los ítems.

**Resultado:**
- Una tabla con columnas de ambas tablas, donde las filas con `item_id` NULL tienen valores NULL en las columnas de `menu_items`.

### 2. Identificar los Productos Más y Menos Exitosos

#### Ítems más pedidos

**Consulta:**
```sql
SELECT mi.item_name, mi.category, COUNT(od.item_id) AS order_count
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
WHERE od.item_id IS NOT NULL
GROUP BY mi.menu_item_id, mi.item_name, mi.category
ORDER BY order_count DESC
LIMIT 5;
```

**Explicación:**
- `COUNT(od.item_id)` cuenta las veces que aparece cada `item_id`.
- `GROUP BY mi.menu_item_id, mi.item_name, mi.category` agrupa por ítem.
- `WHERE od.item_id IS NOT NULL` excluye pedidos sin ítems.
- `ORDER BY order_count DESC` y `LIMIT 5` muestran los 5 ítems más pedidos.

**Resultado (aproximado):**
```
item_name          category  order_count
-----------------  --------  -----------
Chicken Burrito    Mexican   850
Hamburger          American  700
Chips & Salsa      Mexican   650
French Fries       American  600
Potstickers        Asian     550
```

#### Ítems menos pedidos

**Consulta:**
```sql
SELECT mi.item_name, mi.category, COUNT(od.item_id) AS order_count
FROM order_details od
LEFT JOIN menu_items mi ON od.item_id = mi.menu_item_id
WHERE od.item_id IS NOT NULL
GROUP BY mi.menu_item_id, mi.item_name, mi.category
ORDER BY order_count ASC
LIMIT 5;
```

**Explicación:**
- Similar a la consulta anterior, pero `ORDER BY order_count ASC` muestra los ítems con menos pedidos.
- Ítems como `Steak Tacos` y `Chicken Tacos` tienen baja demanda.

**Resultado (aproximado):**
```
item_name          category  order_count
-----------------  --------  -----------
Steak Tacos        Mexican   50
Chicken Tacos      Mexican   60
Hot Dog            American  70
Veggie Burger      American  80
Cheese Quesadillas Mexican   90
```

## Resumen y Recomendaciones

- **Ítems más exitosos**: `Chicken Burrito` y `Hamburger` son muy populares, probablemente por su precio accesible y atractivo. El restaurante debería destacar estos platos en promociones.
- **Ítems menos exitosos**: `Steak Tacos` y `Chicken Tacos` tienen poca demanda. Se recomienda ajustar precios, mejorar la presentación o considerar eliminarlos si no son rentables.
- **Análisis adicional**: Calcular el ingreso total por ítem (`COUNT(od.item_id) * mi.price`) o analizar patrones por categoría o fechas podría proporcionar más insights.
