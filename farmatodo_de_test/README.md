![alt text](https://upload.wikimedia.org/wikipedia/commons/2/27/Farmatodo_logo.svg)

---
# 🧪 Test para Data Engineer Junior - Databricks Community Edition

## 📊 Resumen del Test
**Entorno:** Databricks Community Edition  
**Duración estimada:** 3 horas ⏱️  
**Escenario:** Transformar una base de datos normalizada de retail farmacéutico en un modelo analítico para inteligencia de negocios

---

## ✅ Prerrequisitos

Antes de comenzar, asegúrate de tener:

1. 🔐 **Cuenta en Databricks Community Edition** 
   - Si no tienes una, regístrate gratis en [community.cloud.databricks.com](https://community.cloud.databricks.com/)
   
2. 💻 **Conocimientos básicos de:**
   - PySpark y SQL
   - Git y GitHub
   - Conceptos de modelado dimensional (hechos y dimensiones)
   - Arquitectura medallion (Bronze → Silver → Gold)

3. 🛠️ **Herramientas recomendadas:**
   - VS Code (opcional) para edición local
   - Cliente Git en tu máquina

---

## 🚀 Configuración Inicial

### 📌 Paso 1: Fork y Configuración del Repositorio

#### 1.1 Hacer Fork del Repositorio
- Ve al repositorio original: [github.com/ftd-farmatodo/ftd-da-databricks-test](https://github.com/ftd-farmatodo/ftd-da-databricks-test/tree/master)
- Haz clic en el botón **"Fork"** en la esquina superior derecha
- Esto creará una copia del repositorio en tu cuenta de GitHub

#### 1.2 Configurar Token de Acceso Personal en GitHub
1. Ve a tu perfil de GitHub → **Settings** → **Developer settings** → **Personal access tokens**
2. Crea un nuevo token con permisos de `repo`
3. **⚠️ Importante:** Guarda este token, lo necesitarás en el siguiente paso

#### 1.3 Configurar Credenciales en Databricks
1. Inicia sesión en [Databricks Community Edition](https://community.cloud.databricks.com/)
2. Ve a **User Settings** (esquina superior derecha) → **Developer** → **Git Integration**
3. Agrega tus credenciales:
   - **Git provider:** GitHub
   - **Username:** Tu usuario de GitHub
   - **Token:** El token personal que creaste en el paso anterior
4. Haz clic en **Save** ✅

#### 1.4 Clonar el Repositorio en Databricks
1. En tu workspace de Databricks, ve a **Repos** en el menú lateral
2. Haz clic en **Add Repo** 
3. Pega la URL de **TU fork** (no el repositorio original):
    https://github.com/<TU_USUARIO>/ftd-da-databricks-test.git

4. Haz clic en **Create Repo** 🎉

---

### 📌 Paso 2: Crear y Cargar la Base de Datos

#### 2.1 Navegar al Generador de Datos
1. En tu repositorio clonado, navega a:
```
📁 farmatodo_de_test/
    📁 resources/
        📁 farmatodo_de_test_pipeline/
            📁 explorations/
                📄 data_generator`
```

#### 2.2 Ejecutar el Notebook de Generación
1. Abre el notebook `data_generator`
2. **🔄 Ejecuta todas las celdas** (Ctrl+Alt+Enter o Run All)
3. Este script creará automáticamente:
   - 🏷️ **9 tablas de referencia** (categorías, marcas, proveedores, etc.)
   - 👥 **6 tablas de entidades** (productos, clientes, tiendas, etc.)
   - 💳 **3 tablas transaccionales** (transaction_headers, transaction_items, inventory)

#### 2.3 Verificar la Creación de Tablas
Ejecuta esta query en una nueva celda SQL para confirmar que todo se creó correctamente:
```sql
-- Verificar que todas las tablas fueron creadas
SHOW TABLES IN operations;
```

## 📝 Prueba Técnica - Tareas a Completar

---
### 🔍 Parte 1: Exploración de Datos y Evaluación de Calidad (20 puntos)

#### 📊 Tarea 1.1: Análisis Inicial (10 puntos)
Crea un nuevo notebook llamado `01_exploracion_datos.py` en la carpeta `explorations/` y realiza:

1. **📈 Estadísticas básicas de cada tabla:**
   ```python
   # Ejemplo de código a implementar
   def analyze_table(table_name):
       df = spark.table(f"operations.{table_name}")
       return {
           "tabla": table_name,
           "filas": df.count(),
           "columnas": len(df.columns),
           "nulos": df.filter(df.columns[0].isNull()).count()
       }
   ```

2. **🔗 Verificación de relaciones entre tablas:**
   - Documentar todas las llaves primarias y foráneas
   - Identificar registros huérfanos

3. **⚠️ Identificación de problemas de calidad:**
   - Valores nulos inesperados
   - Duplicados
   - Valores fuera de rango


---

### 🔄 Parte 2: Transformación de Capa Bronze a Silver (35 puntos)

#### 🏗️ Tarea 2.1: Crear Vista Desnormalizada de Transacciones (20 puntos)

Crea un nuevo archivo SQL en `transformations/silver/` llamado `silver_transactions.sql`:

```sql
-- transformations/silver/silver_transactions.sql
CREATE OR REFRESH MATERIALIZED VIEW operations.silver_transactions AS
-- Tu código aquí
```

**📌 Puntos clave a recordar:**
- ✅ Manejar customer_ids NULL (clientes walk-in)
- ✅ Usar solo precios actuales (is_current = true)
- ✅ Limpiar cantidades negativas
- ✅ Eliminar transacciones duplicadas

#### 💰 Tarea 2.2: Agregar Métricas Calculadas (15 puntos)

Agrega las siguientes columnas calculadas:
- 🧮 `line_subtotal`: Subtotal de línea
- 💸 `discount_amount`: Monto de descuento
- 📊 `line_total`: Total de línea
- 🏦 `tax_amount`: Monto de impuesto
- 💵 `final_amount`: Monto final
- 📈 `gross_profit`: Ganancia bruta
- 📉 `profit_margin_pct`: Porcentaje de margen

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

#### Resultado esperado

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

---

### ⭐ Parte 3: Capa Silver a Gold - Modelo Analítico (45 puntos)

#### 📊 Tarea 3.1: Crear Tabla de Hechos de Ventas (15 puntos)

Crea `transformations/gold/fact_sales.sql`:

```sql
-- transformations/gold/fact_sales.sql
CREATE OR REFRESH MATERIALIZED VIEW operations.gold_fact_sales AS
-- Implementa la tabla de hechos según las especificaciones
```

#### 🎯 Tarea 3.2: Crear Tabla Resumen de Ventas (15 puntos)

Crea `transformations/gold/sales_summary.sql` con agregaciones por tienda y producto.

**🎯 Objetivo:** Crear una tabla que responda preguntas de negocio como:
- ¿Cuáles son los productos más vendidos por tienda?
- ¿Qué tienda tiene mejor margen de ganancia?
- ¿Cuál es el ticket promedio por tienda?

#### 📐 Tarea 3.3: Crear Tablas de Dimensiones (15 puntos)

Crea los siguientes archivos en `transformations/gold/`:
- 📦 `dim_product.sql` - Dimensión de productos
- 🏪 `dim_store.sql` - Dimensión de tiendas  
- 👥 `dim_customer.sql` - Dimensión de clientes
- 📅 `dim_date.sql` - Dimensión de fecha (ya proporcionada como ejemplo)

---

#### Resultados esperado

#### 1. Tabla: `sales_summary`

##### Estructura de Columnas

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

---

#### 2. Tabla: `gold_dim_product`

##### Estructura de Columnas

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

---

#### 3. Tabla: `gold_dim_store`

##### Estructura de Columnas

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

#### 4. Tabla: `gold_dim_customer`

##### Estructura de Columnas

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

#### 5. Tabla: `gold_dim_date`

##### Estructura de Columnas

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

---

## 📤 Entrega del Test

### 1️⃣ Guardar y Commitear Cambios
```bash
# En tu workspace de Databricks
git add .
git commit -m "feat: Completar test de Data Engineer Junior"
git push origin main
```

### 2️⃣ Verificar tu Trabajo
Asegúrate de que:
- ✅ Todos los notebooks se ejecutan sin errores
- ✅ Las tablas gold están creadas correctamente
- ✅ Los resultados tienen sentido desde el punto de vista del negocio

### 3️⃣ Enviar el Link
Envía el enlace de tu repositorio fork al equipo de reclutamiento de Farmatodo.

---

## 💡 Tips y Mejores Prácticas

### 🎯 Para Obtener Mejores Resultados:

1. **📝 Documenta tu código:**
   - Explica tu razonamiento en comentarios
   - Justifica decisiones técnicas importantes

2. **🔍 Valida tus resultados:**
   - Verifica que los totales cuadren entre capas
   - Asegúrate de no perder datos en las transformaciones

3. **⚡ Optimiza el rendimiento:**
   - Usa particionamiento cuando sea apropiado
   - Considera el uso de caché para tablas frecuentemente accedidas

4. **🎨 Presenta resultados claros:**
   - Crea visualizaciones simples para mostrar insights clave
   - Incluye métricas de negocio relevantes

### ⚠️ Errores Comunes a Evitar:

- ❌ No manejar valores NULL correctamente
- ❌ Olvidar filtrar registros duplicados
- ❌ No validar la integridad referencial
- ❌ Ignorar los problemas de calidad de datos

---

## 🆘 ¿Necesitas Ayuda?

### 📚 Recursos Útiles:
- [Documentación de Databricks SQL](https://docs.databricks.com/sql/language-manual/index.html)
- [PySpark API Reference](https://spark.apache.org/docs/latest/api/python/)
- [Guía de Modelado Dimensional](https://www.kimballgroup.com/data-warehouse-business-intelligence-resources/kimball-techniques/dimensional-modeling-techniques/)

### ❓ FAQs:

**P: ¿Puedo usar Python en lugar de SQL?**  
R: Sí, puedes usar PySpark donde prefieras, pero SQL es recomendado para las transformaciones.

**P: ¿Qué pasa si encuentro datos inconsistentes?**  
R: Documéntalo y explica cómo lo resolviste. Esto es parte de la evaluación.

**P: ¿Puedo crear tablas adicionales?**  
R: Sí, si lo consideras necesario para mejorar el modelo.

---

**¡Mucho éxito! 🚀 Estamos emocionados de ver tu solución.**

*Recuerda: La calidad es más importante que la velocidad. Tómate el tiempo necesario para entregar tu mejor trabajo.* 💪
```