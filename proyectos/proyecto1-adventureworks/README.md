💰 Análisis de Desempeño Financiero con SQL · AdventureWorks

Bootcamp: TripleTen · Data AnalyticsSprint: 3 — Explorar KPIs con SQLHerramientas: SQL · Amazon QuickSight · Google Sheets

📋 Contexto del proyecto

Soy analista en AdventureWorks. El director financiero quiere saber en qué mercados se generan más ingresos y rentabilidad para decidir dónde invertir el próximo presupuesto de marketing.

Con datos de órdenes, productos, territorios y campañas, mi tarea fue preparar un análisis que mostrara prioridades de mercado, optimización de presupuesto y rentabilidad por país.

Preguntas clave del negocio:
	∙	¿Cuánto estamos ganando por país?
	∙	¿Qué tan rentable es cada mercado considerando los gastos de marketing?

  🗂 Dataset
  
Tablas utilizadas:

ventas_2017 - Transacciones de líneas de pedido (2017)
productos - Catálogo con costo y precio unitario
productos_categorias - Jerarquía categoría/subcategoría
territorios - Mapa de ClaveTerritorio → país y continente
campanas - Gasto de marketing por territorio

🔍 Mi análisis — Proceso paso a paso

Parte 1: Explorar el esquema

Imprimí los primeros 10 registros de cada tabla para entender la estructura de datos e identificar las claves de unión (ClaveProducto, ClaveTerritorio, IDCampana).

SELECT * FROM ventas_2017 LIMIT 10;

SELECT * FROM productos LIMIT 10;

SELECT * FROM productos_categorias LIMIT 10;

SELECT * FROM territorios LIMIT 10;

SELECT * FROM campanas LIMIT 10;

Parte 2: Extraer y limpiar datos

Construí una tabla base (ventas_clean) uniendo las 4 tablas con JOINs y calculando dos columnas nuevas:

∙	ingreso_total = precio × cantidad
∙	costo_total = costo × cantidad

Usé COALESCE para reemplazar valores nulos por 0 y evitar errores en los cálculos.

SELECT
    v.numero_pedido,
    v.clave_producto,
    p.nombre_producto,
    pc.clave_categoria,
    COALESCE(p.precio_producto, 0) AS precio_producto,
    COALESCE(v.cantidad_pedido, 0) AS cantidad_pedido,
    COALESCE(p.costo_producto, 0)  AS costo_producto,
    t.pais,
    t.continente,
    v.clave_territorio,
    COALESCE(p.precio_producto, 0) * COALESCE(v.cantidad_pedido, 0) AS ingreso_total,
    COALESCE(p.costo_producto, 0)  * COALESCE(v.cantidad_pedido, 0) AS costo_total
FROM ventas_2017 AS v
JOIN productos AS p ON v.clave_producto = p.clave_producto
LEFT JOIN productos_categorias AS pc ON p.clave_subcategoria = pc.clave_subcategoria
LEFT JOIN territorios AS t ON v.clave_territorio = t.clave_territorio;

Parte 3: Calcular KPIs financieros

Calculé los indicadores clave por país:

Beneficio Bruto Ingresos − Costos
Margen % (Ingresos − Costos) / Ingresos × 100
ROI % (Ingresos − Costos) / Costo Campaña × 100

SELECT
    p.pais,
    p.clave_territorio,
    SUM(p.ingresos)::integer AS ingresos,
    SUM(p.costos)::integer AS costos,
    COALESCE(SUM(c.costo_campana), 0)::integer AS costo_campana,
    (SUM(p.ingresos) - SUM(p.costos))::integer AS beneficio_bruto,
    (SUM(p.ingresos) - SUM(p.costos)) * 100.0 / NULLIF(SUM(p.ingresos), 0) AS margen_pct,
    (SUM(p.ingresos) - SUM(p.costos)) * 100.0 / NULLIF(SUM(c.costo_campana), 0) AS roi_pct
FROM pais_ingreso_costo AS p
LEFT JOIN pais_campanas AS c ON p.clave_territorio = c.clave_territorio
GROUP BY p.pais, p.clave_territorio
ORDER BY p.clave_territorio, ingresos, costos;

Parte 4: Validar resultados y QA
Verifiqué la calidad de los datos:

∙	Validé que no hubiera NULLs en claves principales


SELECT
    SUM(CASE WHEN numero_pedido IS NULL THEN 1 ELSE 0 END) AS nulos_numero_pedido,
    SUM(CASE WHEN clave_producto IS NULL THEN 1 ELSE 0 END) AS nulos_clave_producto,
    SUM(CASE WHEN clave_territorio IS NULL THEN 1 ELSE 0 END) AS nulos_clave_territorio
FROM ventas_2017;

∙	Comprobé que no existieran precios negativos (productos_precio_no_valido = 0)

SELECT
    COUNT(*) AS productos_precio_no_valido
FROM productos
WHERE precio_producto < 0;
-- Resultado esperado: 0

∙	 Confirmé que los totales por país cuadraran con los totales generales

-- QA 3: Validar que totales por país cuadren con total general
SELECT
    SUM(ingreso_total) AS total_ingresos_general,
    SUM(costo_total)   AS total_costos_general
FROM ventas_clean;

-- Comparar contra suma por pais
SELECT
    pais,
    SUM(ingreso_total) AS ingresos_por_pais,
    SUM(costo_total)   AS costos_por_pais
FROM ventas_clean
GROUP BY pais
ORDER BY ingresos_por_pais DESC;

📊 Conclusiones principales

Resultados finales por mercado:

País Ingresos Beneficio Bruto Margen % ROI %

United States $3,353,940 $1,454,469 43.37% 75.75%
Australia $2,532,003 $1,057,045 41.75% 49.16%
Germany $1,071,460 $460,165 42.95% 20.31%
United Kingdom $1,189,637 $508,128 42.71% 22.05%
France $924,317 $396,520 42.90% 17.96%
Canada $710,205 $317,879 44.76% 17.43%

Hallazgos clave (metodología C→F→I):

1. Liderazgo en Rentabilidad — Estados Unidos
Estados Unidos es el mercado más robusto con un ROI de 75.75% y margen bruto de 43.37%. Es el mercado con mayor madurez y base de clientes consolidada.

3. Desempeño Sólido — Australia
Australia presenta un ROI de 49.16%, el segundo más alto. Mercado con alto potencial de crecimiento y eficiencia operativa saludable.

5. Punto Crítico — Canadá
Canadá muestra el ROI más bajo (17.43%). Se recomienda revisar la estructura de costos y redistribuir parte del presupuesto de campañas hacia mercados con mayor retorno.

💡 Aprendizajes

∙	El uso de COALESCE es fundamental para evitar errores de cálculo en datos reales con valores nulos.

∙	NULLIF protege contra divisiones por cero al calcular márgenes y ROI.

∙	La validación QA antes de presentar resultados es esencial — un solo NULL en una clave puede distorsionar todos los totales.

∙	El ROI y el margen cuentan historias diferentes: un mercado puede tener alto margen pero bajo ROI si la inversión en campañas es desproporcionada.

Proyecto desarrollado en TripleTen · Data Analytics Bootcamp · 2026

