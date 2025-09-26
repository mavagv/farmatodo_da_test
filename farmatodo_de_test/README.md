![alt text](https://upload.wikimedia.org/wikipedia/commons/2/27/Farmatodo_logo.svg)

---
# Test para Data Engineer Junior - Databricks Free Edition

## Resumen del Test
**Entorno:** Databricks Free Edition
**Escenario:** Transformar una base de datos normalizada de retail farmacéutico en un modelo analítico para inteligencia de negocios

---

## Prerrequisitos

1. Acceso a Databricks Free Edition
2. Conocimiento básico de PySpark, SQL, Git
3. Comprensión de conceptos de modelado dimensional

---

## Configuración Inicial

### Paso 1: Configurar repo en el workspace de Databricks

- Realizar un fork del [repositorio](https://github.com/ftd-farmatodo/ftd-da-databricks-test/tree/master) del test
- Configurar tus credenciales de [github con Databricks](https://docs.databricks.com/aws/en/repos/get-access-tokens-from-git-provider) 
- Clonar tu nuevo repositorio al workspace de Databricks

### Paso 2: Crear y Cargar la Base de Datos

Ejecuta el notebook proporcionado para generar la base de datos normalizada con 18 tablas en el esquema `./farmatodo_de_test/resources/farmatodo_de_test_pipeline/explorations/data_generator`

El script creará:
- 9 tablas de referencia (categorías, marcas, proveedores, etc.)
- 6 tablas de entidades (productos, clientes, tiendas, etc.)
- 3 tablas transaccionales (transaction_headers, transaction_items, inventory)

Verifica que todas las tablas estén cargadas:
```sql
SHOW TABLES IN operations
```

---

## Prueba Tecnica

### Parte 1: Exploración de Datos y Evaluación de Calidad (20 puntos)

### Parte 2: Transformación de Capa Bronze a Silver (35 puntos)

#### Tarea 2.1: Crear Vista Desnormalizada de Transacciones (20 puntos)

Une las tablas normalizadas para crear la tabla `silver_transactions` con la siguiente estructura:

| Nombre de Columna | Descripción | Origen |
|-------------------|-------------|---------|
| transaction_id | Identificador único de transacción | transaction_headers |
| item_id | Identificador único de línea | transaction_items |
| transaction_date | Fecha de transacción | transaction_headers |
| transaction_time | Hora de transacción | transaction_headers |
| store_id | Identificador de tienda | transaction_headers |
| store_name | Nombre de tienda (distrito) | stores.district |
| store_type | Tipo de tienda | store_types.type_name |
| city | Nombre de ciudad | cities.city_name |
| country | Código de país | cities.country_code |
| customer_id | Identificador de cliente | transaction_headers |
| customer_name | Nombre completo (nombre + apellido) | customers |
| customer_type | Clasificación de cliente | customer_types.type_name |
| product_id | Identificador de producto | transaction_items |
| product_name | Nombre de producto | products |
| sku | SKU del producto | products |
| category | Nombre de categoría | categories |
| subcategory | Nombre de subcategoría | subcategories |
| brand | Nombre de marca | brands |
| supplier | Nombre de proveedor | suppliers |
| quantity | Cantidad vendida | transaction_items |
| unit_cost | Costo por unidad | product_prices (actual) |
| unit_price | Precio de venta por unidad | product_prices (actual) |
| discount_percent | Descuento aplicado | transaction_items |
| tax_rate | Tasa de impuesto del país | countries |
| payment_method | Tipo de pago | payment_methods |

**Requerimientos:**
- Manejar customer_ids NULL (clientes sin registro)
- Usar solo precios actuales (is_current = true)
- Limpiar cantidades negativas (convertir a positivo o filtrar)
- Eliminar transacciones duplicadas

#### Tarea 2.2: Agregar Métricas Calculadas (15 puntos)


```sql
-- Cálculos requeridos:
    quantity * unit_price as line_subtotal,
    (quantity * unit_price) * (discount_percent / 100.0) as discount_amount,
    (quantity * unit_price) * (1 - discount_percent / 100.0) as line_total,
    (quantity * unit_price) * (1 - discount_percent / 100.0) * tax_rate as tax_amount,
    (quantity * unit_price) * (1 - discount_percent / 100.0) * (1 + tax_rate) as final_amount,
    (unit_price - unit_cost) * quantity as gross_profit,
    ((unit_price - unit_cost) / NULLIF(unit_price, 0)) * 100 as profit_margin_pct
```

**Entregable:** Guardar como `operations.silver_transactions`


### Parte 3: Capa Silver a Gold - Modelo Analítico (45 puntos)
#### Tarea 3.1: Crear Tabla de Hechos de Ventas (15 puntos)
Crear `gold_fact_sales` con una fila por artículo de transacción:

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| store_id | STRING | NO | Identificador único de tienda (ej: 'VE001', 'CO002') |
| store_name | STRING | NO | Nombre del distrito de la tienda |
| city | STRING | NO | Ciudad donde está ubicada la tienda |
| country | STRING | NO | Código del país (VE, CO, AR) |
| product_id | STRING | NO | Identificador único del producto (ej: 'PROD0001') |
| product_name | STRING | NO | Nombre descriptivo del producto |
| category | STRING | NO | Categoría principal del producto |
| brand | STRING | NO | Marca del producto |
| total_quantity | INTEGER | NO | Suma total de unidades vendidas |
| total_sales | DECIMAL(10,2) | NO | Monto total de ventas en moneda local |
| total_cost | DECIMAL(10,2) | NO | Costo total de los productos vendidos |
| total_profit | DECIMAL(10,2) | NO | Ganancia bruta (ventas - costo) |
| profit_margin_pct | DECIMAL(5,2) | NO | Porcentaje de margen de ganancia promedio |
| num_transactions | INTEGER | NO | Cantidad de transacciones únicas |
| num_customers | INTEGER | NO | Cantidad de clientes únicos |
| avg_sale_price | DECIMAL(10,2) | NO | Precio de venta promedio |
| first_sale_date | DATE | NO | Primera fecha de venta del producto en la tienda |
| last_sale_date | DATE | NO | Última fecha de venta del producto en la tienda |

#### Características de la Tabla
- **Llave Primaria Compuesta**: (store_id, product_id)
- **Granularidad**: Una fila por cada combinación única de tienda-producto
- **Registros Esperados**: 600-800 filas aproximadamente
- **Ordenamiento**: Por defecto ordenado por total_sales DESC
---

#### Tarea 3.2 Crear Tablas de Dimensiones (30 puntos)

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| sale_id | STRING | NO | Identificador único de la línea de venta |
| transaction_id | STRING | NO | Identificador de la transacción |
| date_key | INTEGER | NO | Llave de fecha en formato YYYYMMDD |
| store_key | STRING | NO | Llave foránea a dim_store |
| product_key | STRING | NO | Llave foránea a dim_product |
| customer_key | STRING | NO | Llave foránea a dim_customer |
| quantity_sold | INTEGER | NO | Cantidad de unidades vendidas |
| unit_cost | DECIMAL(10,2) | NO | Costo unitario del producto |
| unit_price | DECIMAL(10,2) | NO | Precio de venta unitario |
| discount_amount | DECIMAL(10,2) | NO | Monto del descuento aplicado |
| tax_amount | DECIMAL(10,2) | NO | Monto del impuesto |
| sale_amount | DECIMAL(10,2) | NO | Monto total de la venta |
| gross_profit | DECIMAL(10,2) | NO | Ganancia bruta de la línea |

---

### 3. Tabla: `gold_dim_product`

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| product_key | STRING | NO | Llave primaria (product_id) |
| product_name | STRING | NO | Nombre del producto |
| sku | STRING | NO | Código SKU |
| category | STRING | NO | Categoría principal |
| subcategory | STRING | NO | Subcategoría |
| brand | STRING | NO | Marca |
| supplier | STRING | NO | Proveedor |
| is_imported | BOOLEAN | NO | Indicador de producto importado |
| requires_prescription | BOOLEAN | NO | Requiere receta médica |
| status | STRING | NO | Estado (Active/Discontinued) |
| unit_cost | DECIMAL(10,2) | NO | Costo unitario actual |
| unit_price | DECIMAL(10,2) | NO | Precio de venta actual |
| profit_margin_pct | DECIMAL(5,2) | YES | Porcentaje de margen |
| price_range | STRING | NO | Rango de precio (Bajo/Medio/Alto/Premium) |

---

### 4. Tabla: `gold_dim_store`

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| store_key | STRING | NO | Llave primaria (store_id) |
| store_name | STRING | NO | Nombre del distrito |
| store_type | STRING | NO | Tipo (Regular/Express/Super) |
| city | STRING | NO | Ciudad |
| country | STRING | NO | País |
| size_m2 | INTEGER | NO | Tamaño en metros cuadrados |
| num_employees | INTEGER | NO | Número de empleados |
| size_category | STRING | NO | Categoría de tamaño (Pequeña/Mediana/Grande) |

---

### 5. Tabla: `gold_dim_customer`

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| customer_key | STRING | NO | Llave primaria (customer_id o 'WALK_IN') |
| customer_name | STRING | NO | Nombre completo |
| customer_type | STRING | NO | Tipo de cliente |
| city | STRING | NO | Ciudad de residencia |
| country | STRING | NO | País de residencia |
| gender | STRING | YES | Género (M/F/O/NULL) |
| age_group | STRING | NO | Grupo de edad |
| is_active | BOOLEAN | NO | Cliente activo |
| loyalty_points | INTEGER | NO | Puntos de lealtad actuales |
| customer_segment | STRING | NO | Segmento (Nuevo/Regular/Premium/VIP/Walk-in) |

---

### 6. Tabla: `gold_dim_date`

#### Estructura de Columnas

| Columna | Tipo de Dato | Nullable | Descripción |
|---------|--------------|----------|-------------|
| date_key | INTEGER | NO | Llave primaria en formato YYYYMMDD |
| full_date | DATE | NO | Fecha completa |
| year | INTEGER | NO | Año |
| quarter | INTEGER | NO | Trimestre (1-4) |
| month | INTEGER | NO | Mes (1-12) |
| month_name | STRING | NO | Nombre del mes |
| week | INTEGER | NO | Semana del año |
| day_of_month | INTEGER | NO | Día del mes |
| day_of_week | INTEGER | NO | Día de la semana (1-7) |
| day_name | STRING | NO | Nombre del día |
| is_weekend | BOOLEAN | NO | Indicador de fin de semana | 