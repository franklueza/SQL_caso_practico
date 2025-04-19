# Análisis SQL para el Restaurante "Sabores del Mundo"

Este documento presenta la solución paso a paso al caso práctico del restaurante "Sabores del Mundo".  

## Contexto
 El restaurante "Sabores del Mundo", es conocido por su auténtica cocina y su ambiente  acogedor.  
 Este restaurante lanzó un nuevo menú a principios de año y ha estado recopilando  información detallada sobre las transacciones de los clientes para identificar áreas de
 oportunidad y aprovechar al máximo sus datos para optimizar las ventas.  

**Objetivo**  
   
 Identificar cuáles son los productos del menú que han tenido más éxito y cuales son los que
 menos han gustado a los clientes

## a) Crear la Base de Datos  

`CREATE DATABASE restaurant;`  
  
El archivo `create_restaurant_db.sql` proporciona el script para crear dos tablas:

- **menu_items**: Contiene información sobre los ítems del menú (`menu_item_id`, `item_name`, `category`, `price`).
- **order_details**: Contiene detalles de los pedidos (`order_details_id`, `order_id`, `order_date`, `order_time`, `item_id`).

**Acción**: Ejecutar el script SQL en PostgreSQL

## b) Explorar la tabla `menu_items`

`SELECT * FROM menu_items;`  
  
Analizamos la tabla `menu_items` para responder las siguientes preguntas sobre los productos del menú.

### 1. Encontrar el número de artículos en el menú

**Consulta:**
```sql
SELECT COUNT(*) AS total_items
FROM menu_items;
```

**Explicación:**
- `COUNT(*)` cuenta todas las filas en la tabla `menu_items`, lo que representa el número total de artículos en el menú

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
SELECT COUNT(*) AS total_platos_americanos
FROM menu_items
WHERE category = 'American';
```

**Explicación:**
- Filtramos los ítems donde `category = 'American'` usando `WHERE`.
- `COUNT(*)` cuenta las filas que cumplen con esta condición.
- Los ítems americanos son: Hamburger, Cheeseburger, Hot Dog, Veggie Burger, Mac & Cheese, French Fries (6 ítems).

**Resultado:**
```
total_platos_americanos
---------------
6
```
**Consulta:**
```sql
SELECT item_name
FROM menu_items
WHERE category = 'American';;
```
```
item_name
---------------
Hamburger
Cheeseburger
Hot Dog
Veggie Burger
Mac & Cheese
French Fries
```

### 4. ¿Cuál es el precio promedio de los platos?

**Consulta:**
```sql
SELECT ROUND(AVG(price), 2) AS precio_promedio
FROM menu_items;
```

**Explicación:**
- `AVG(price)` calcula el promedio de la columna `price`.
- `ROUND(..., 2)` redondea el resultado a 2 decimales.

**Resultado:**
```
precio_promedio
-------------
13.29
```

## c) Explorar la Tabla `order_details`
`SELECT * FROM order_details;`
  
Analizamos los datos de los pedidos en la tabla `order_details`.

### 1. ¿Cuántos pedidos únicos se realizaron en total?

**Consulta:**
```sql
SELECT COUNT(DISTINCT order_id) AS total_pedidos_unicos
FROM order_details;
```

**Explicación:**
- `DISTINCT order_id` elimina duplicados para contar solo pedidos únicos.
- `COUNT(DISTINCT order_id)` cuenta el número de `order_id` distintos.
**Resultado:**
```
 total_pedidos_unicos
-------------
5370
```

### 2. ¿Cuáles son los 5 pedidos que tuvieron el mayor número de artículos?

**Consulta:**
```sql
SELECT order_id, COUNT(*) AS total_pedidos
FROM order_details
GROUP BY order_id
ORDER BY total_pedidos DESC
LIMIT 5;
```

**Explicación:**
- `COUNT(*)` cuenta los ítems por pedido.
- `GROUP BY order_id` agrupa por order_id
- `ORDER BY total_pedidos DESC` y `LIMIT 5` muestran los 5 pedidos con más ítems.

**Resultado (aproximado):**
```
order_id  item_count
--------  ----------
440	      14
2675	      14
3473	      14
4305	      14
443	      14
```

### 3. ¿Cuándo se realizó el primer y el último pedido?

**Consulta:**
```sql
SELECT 
	MIN(order_date) AS primer_pedido,
	MAX(order_date) AS ultimo_pedido
	FROM order_details;
```

**Explicación:**
- `MIN(order_date)` encuentra la fecha más temprana.
- `MAX(order_date)` encuentra la fecha más reciente.

**Resultado:**
```
primer_pedido  ultimo_pedido
------------  ------------
2023-01-01    2023-03-31
```

### 4. ¿Cuántos pedidos se hicieron entre el '2023-01-01' y el '2023-01-05'?

**Consulta:**
```sql
SELECT COUNT(DISTINCT order_id) AS periodo_pedidos
FROM order_details
WHERE order_date BETWEEN '2023-01-01' AND '2023-01-05'
```

**Explicación:**
- `WHERE order_date BETWEEN '2023-01-01' AND '2023-01-05'` filtra los pedidos en el rango de fechas.
- `COUNT(DISTINCT order_id)` cuenta los pedidos únicos..

**Resultado:**
```
periodo_pedidos
--------------
308
```
```sql
SELECT COUNT(*) 
FROM order_details
WHERE order_date BETWEEN '2023-01-01' AND '2023-01-05';
```
```
orders_between
--------------
702
```
## d) Usar ambas tablas para conocer la reacción de los Clientes

### 1. Realizar un left join entre entre order_details y menu_items con el identificador item_id(tabla order_details) y menu_item_id(tabla menu_items).

**Consulta:**
```sql
SELECT 
    o.order_details_id,
    o.order_id,
    o.order_date,
    o.order_time,
    o.item_id,
    m.menu_item_id,
    m.item_name,
    m.category,
    m.price
FROM order_details o
LEFT JOIN menu_items m
ON o.item_id = m.menu_item_id;
```

**Explicación:**
- `LEFT JOIN` incluye todas las filas de `order_details`, incluso si no hay coincidencia en `menu_items` (por ejemplo, cuando `item_id` es NULL).
- `ON od.item_id = mi.menu_item_id` vincula las tablas.
- Esta consulta combina los datos de pedidos con los detalles de los ítems.

## e) El  objetivo es identificar 5 puntos clave que puedan ser de utilidad para los dueños del  restaurante en el lanzamiento de su nuevo menú.
### La siguiente consulta calcula la popularidad, ingresos y distribución por categoría de los primeros 5 artículos:

**Consulta:**
```sql
SELECT 
    m.item_name,
    m.category,
    COUNT(o.item_id) AS recuento,
    ROUND(COUNT(o.item_id) * m.price, 2) AS ganancia_total,
    ROUND(m.price, 2) AS precio_articulo
FROM order_details o
LEFT JOIN menu_items m
ON o.item_id = m.menu_item_id
WHERE o.item_id IS NOT NULL
GROUP BY m.menu_item_id, m.item_name, m.category, m.price
ORDER BY recuento DESC   -- podemos cambiar a ASC, para ver los 5 artículos menos pedidos
LIMIT 5;
```

**Resultados:**
```
item_name          category  order_count  ganancia_total  precio_articulo
-----------------  --------  -----------  -------------  ----------
Hamburger	   American	622	    8054.90	   12.95
Edamame 	   Asian	620	    3100.00	   5.00
Korean Beef Bowl   Asian	588	    10554.60	   17.95
Cheeseburger	   American	583	    8132.85	   13.95
French Fries	   American	571	    3997.00	   7.00
```

```
item_name          category  order_count  ganancia_total  precio_articulo
-----------------  --------  -----------  -------------  ----------
Chicken Tacos	     Mexican	123	    1469.85	   11.95
Potstickers	     Asian	205	    1845.00	   9.00
Cheese Lasagna	     Italian	207	    3208.50	   15.50
Steak Tacos	     Mexican	214	    2985.30	   13.95
Cheese Quesadillas   Mexican	233	    2446.50	   10.50
```

## Resumen y Recomendaciones

- **Los artículos más exitosos**: `Hamburger` y `Edamame` son muy populares, probablemente por su precio accesible y atractivo. El restaurante debería destacar estos platos en promociones.
- **Ítems menos exitosos**: `Chicken Tacos` y `Potstickers` tienen poca demanda. Se recomienda ajustar precios, mejorar la presentación o considerar eliminarlos si no son rentables.
- **Ganancias**: El artículo en la pposición 3 más popular que ha generado más ganancias es el `Korean Beef Bowl` perteneciente a la categoria `Asian` siendo uno  de los más caros.
