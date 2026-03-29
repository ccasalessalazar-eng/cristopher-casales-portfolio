📊 Análisis de Embudo y Retención · MercadoLibre

Bootcamp: TripleTen · Data AnalyticsSprint: 4 — Análisis de Embudo y RetenciónHerramientas: SQL · CTEs · Google Sheets

📋 Contexto del proyecto

Soy analista de producto en MercadoLibre, dentro del equipo de Crecimiento y Retención. El director de producto planteó el reto:

“Necesitamos entender en qué etapa del proceso perdemos usuarios y cómo podemos mejorar su retención a lo largo del tiempo.”

Mi misión fue usar SQL para mapear el embudo de conversión completo, identificar los principales puntos de fuga y evaluar la retención de usuarios por cohortes, proponiendo mejoras accionables basadas en datos.

Preguntas clave del negocio:

∙	Entre el 01/01/2025 y el 08/31/2025, ¿cuál es la tasa de conversión entre cada etapa del embudo?
	
∙	¿En qué paso se observa la mayor caída de usuarios?

∙	Para usuarios registrados entre el 01/01/2025 y el 06/01/2025, ¿cuál es la retención en D7, D14, D21, D28?

🗂 Dataset


Eventos del embudo (Macro Journey)

🔍 Mi análisis — Proceso paso a paso

Parte 1: Explorar el esquema
Exploré las tablas para entender la estructura de datos, tipos de columnas y secuencia de eventos disponibles.

SELECT * FROM mercadolibre_funnel LIMIT 5;
SELECT * FROM mercadolibre_retention LIMIT 5;
SELECT DISTINCT event_name FROM mercadolibre_funnel ORDER BY event_name;


Parte 2: Construir el embudo de conversión

Construí el embudo con CTEs encadenados — uno por etapa — usando DISTINCT user_id para evitar duplicados, y uní todo con LEFT JOIN desde first_visit.

Parte 3: Analizar retención y cohortes

Calculé la retención por cohorte mensual (D7, D14, D21, D28) usando DATE_TRUNC para agrupar por mes de registro y CASE WHEN para contar usuarios activos acumulados.

WITH cohort AS (
    SELECT
        user_id,
        TO_CHAR(DATE_TRUNC('month', MIN(signup_date)), 'YYYY-MM') AS cohort
    FROM mercadolibre_retention
    GROUP BY user_id
),
activity AS (
    SELECT
        r.user_id,
        c.cohort,
        r.day_after_signup,
        r.active
    FROM mercadolibre_retention r
    LEFT JOIN cohort c ON r.user_id = c.user_id
    WHERE r.activity_date BETWEEN '2025-01-01' AND '2025-08-31'
)
SELECT
    cohort,
    ROUND(COUNT(DISTINCT CASE WHEN day_after_signup >= 7  AND active = 1 THEN user_id END) * 100.0 /
          NULLIF(COUNT(DISTINCT user_id), 0), 1) AS retention_d7_pct,
    ROUND(COUNT(DISTINCT CASE WHEN day_after_signup >= 14 AND active = 1 THEN user_id END) * 100.0 /
          NULLIF(COUNT(DISTINCT user_id), 0), 1) AS retention_d14_pct,
    ROUND(COUNT(DISTINCT CASE WHEN day_after_signup >= 21 AND active = 1 THEN user_id END) * 100.0 /
          NULLIF(COUNT(DISTINCT user_id), 0), 1) AS retention_d21_pct,
    ROUND(COUNT(DISTINCT CASE WHEN day_after_signup >= 28 AND active = 1 THEN user_id END) * 100.0 /
          NULLIF(COUNT(DISTINCT user_id), 0), 1) AS retention_d28_pct
FROM activity
GROUP BY cohort
ORDER BY cohort;


📊 Conclusiones principales

Resultados del embudo de conversión

Resultados de retención por cohorte

Hallazgos clave (metodología C→F→I):

1. El “agujero negro” del embudo — select_item → add_to_cart
  
El punto de mayor caída está en la transición de interés a intención de compra: se pierde el 85.7% de los usuarios. Los usuarios ven productos pero no los agregan al carrito, lo que sugiere fricción en la página de detalle del producto (precio, tiempos de entrega, confianza).

2. Fricción en pagos — México
   
México muestra una caída más pronunciada en add_payment_info, lo que indica problemas con los métodos de pago disponibles o falta de confianza en las opciones locales.

3. Mejor retención — Cohortes de enero y febrero (2025)

Los usuarios registrados en enero y febrero muestran las mejores tasas de retención a D28. Las cohortes de junio y julio presentan caídas más drásticas en D7 y D14, lo que sugiere que el tráfico captado a inicio de año tiene mejor “Product-Market Fit”.

Implicaciones y recomendaciones:

∙	Optimizar la página de detalle de producto (PDP): auditar claridad de costos de envío y tiempos de entrega

∙	Integrar más métodos de pago locales en México y países con alta caída en add_payment_info

∙	Implementar campañas de re-engagement (D10-D14) para cohortes con caída acelerada

💡 Aprendizajes

∙	Los CTEs encadenados con LEFT JOIN desde first_visit son la forma más limpia de construir embudos multietapa en SQL.

∙	NULLIF es esencial al calcular porcentajes — sin él, la división por cero rompe la query.

∙	El análisis de cohortes revela patrones invisibles en el promedio general: una cohorte puede tener buena retención D7 pero colapsar en D14.

∙	La segmentación por país expone problemas específicos de mercado que los datos globales ocultan.

Proyecto desarrollado en TripleTen · Data Analytics Bootcamp · 2026


