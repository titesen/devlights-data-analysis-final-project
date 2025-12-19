# 📘 Documentación técnica: Chinook Strategy Command Center

**Business Intelligence & Data Engineering Project**

## 1. Resumen Ejecutivo

El **Chinook Strategy Command Center** es una solución de Inteligencia de Negocios (BI) diseñada para transformar los datos transaccionales de una tienda de música digital global en activos estratégicos. Este proyecto aborda la problemática de la "ceguera operativa", donde la abundancia de datos crudos no se traducía en información accionable.

A través de un pipeline de Ingeniería de Datos (ETL), hemos migrado de un enfoque reactivo a uno proactivo, permitiendo a la gerencia visualizar la salud financiera, segmentar clientes por valor y optimizar el catálogo de productos en tiempo real.

---

## 2. Marco Estratégico (OKRs y KPIs)

El proyecto se rige bajo el siguiente marco de Objetivos y Resultados Clave (OKRs):

- **Objetivo (O):** Democratizar el acceso a insights estratégicos para maximizar la rentabilidad y retención de clientes.

- **Key Result 1 (KR):** Reducir el tiempo de obtención de métricas financieras de horas a segundos mediante un modelo OLAP.

- **Key Result 2 (KR):** Identificar y aislar al top 20% de clientes de alto valor (Segmentación RFM) para estrategias de retención.

- **Key Result 3 (KR):** Detectar ineficiencias en el inventario digital (formatos y géneros de baja rotación).

---

## 3. Arquitectura de Datos: De OLTP a OLAP

Uno de los pilares técnicos de este proyecto fue la transformación del modelo de datos para optimizar el rendimiento analítico.

### 3.1 El Punto de Partida: Modelo OLTP (Transaccional)

- **Estructura:** Base de datos altamente normalizada (3ra Forma Normal).

- **Características:** Diseñada para la integridad de datos y escrituras rápidas (INSERT/UPDATE/DELETE).

- **Problema Analítico:** Para responder una pregunta simple como *"¿Cuánto vendió Jane Peacock en Brasil en 2023?"*, se requerían más de 5 `JOINs` complejos entre tablas dispersas (`Invoice`, `InvoiceLine`, `Customer`, `Employee`, `Track`, `Album`, `Artist`). Esto generaba latencia y complejidad en las consultas.

### 3.2 La Solución: Modelo OLAP (Analítico)

- **Transformación (ETL):** Se creó un **Data Warehouse** utilizando un **Esquema Estrella (Star Schema)** dentro del esquema `analytics`.

- **Cambios Clave:**

- **Desnormalización:** Se redujo la complejidad uniendo tablas relacionadas. Por ejemplo, `dim_track` ahora contiene datos del track, álbum, artista y género en una sola fila.

- **Tabla de Hechos (`fact_sales`):** Centraliza todas las métricas numéricas (ingresos, cantidad) y las claves foráneas.

- **Tablas de Dimensiones (`dim_...`):** Proporcionan el contexto (Quién, Cuándo, Qué, Dónde).

- **Beneficio:** Las consultas ahora son directas y eficientes, permitiendo el filtrado dinámico en el dashboard sin impactar el rendimiento.

---

## 4. Desglose del Dashboard y Análisis Visual

El dashboard se divide en cuatro secciones estratégicas, cada una diseñada para responder preguntas de negocio específicas.

### SECCIÓN 1: Resumen Financiero (The Pulse)

**Objetivo:** Proporcionar conciencia situacional inmediata sobre la salud macroeconómica del negocio.

- **KPI 1: Ingresos Totales (Total Revenue)**

- *Definición:* Suma total de ventas netas.

- *Insight:* Es la "North Star Metric" que indica el volumen de negocio.

- **KPI 2: Ticket Promedio (AOV - Average Order Value)**

- *Definición:* Ingresos Totales / Cantidad de Transacciones Únicas.

- *Pregunta que responde:* ¿Qué tan eficientes somos en cada venta? ¿Están funcionando las estrategias de up-selling?

- **KPI 3: Cantidad de Clientes VIP (Campeones)**

- *Definición:* Conteo en tiempo real de clientes segmentados como "Campeones" por el algoritmo RFM.

- *Insight:* Monitor de retención. Una caída aquí es una alerta roja inmediata.

### SECCIÓN 2: Inteligencia de Clientes (Matriz RFM)

**Objetivo:** Pasar de ver "promedios" a entender comportamientos individuales. Se utilizó un algoritmo SQL para calcular puntajes de **Recencia** (días desde última compra), **Frecuencia** y **Valor Monetario**.

- **Gráfico: Matriz de Valor (Scatter Plot)**

- *Eje X (Recencia):* Riesgo de abandono (más a la derecha = más tiempo sin comprar).

- *Eje Y (Monetario):* Valor del cliente (más arriba = más dinero gastado).

- *Tamaño de Burbuja:* Frecuencia de compra.

- *Color:* Segmento Automático (Campeones, Leales, En Riesgo, Perdidos).

- *Insight:* Permite identificar visualmente a los clientes "Ballena" (VIPs) y diferenciarlos de los clientes que gastaron mucho una vez pero nunca volvieron.

- **Gráfico: Distribución de Segmentos (Donut Chart)**

- *Objetivo:* Entender la composición de la base de clientes.

- *Insight:* Si el segmento "Perdidos" supera el 30-40%, la empresa tiene un problema grave de retención (Churn).

### SECCIÓN 3: Operaciones y Mercado Global

**Objetivo:** Optimizar la logística digital y enfocar esfuerzos de marketing geográfico.

- **Gráfico: Mapa de Calor de Ventas (Geo Map)**

- *Transformación:* Se normalizaron datos geográficos (ej. 'USA' -> 'United States') para asegurar la correcta renderización ISO.

- *Insight:* Identificación de mercados saturados vs. mercados emergentes con alto ROI.

- **Gráfico: Pareto de Formatos de Archivo (Barras)**

- *Pregunta que responde:* ¿Qué formatos técnicos generan ingresos reales?

- *Insight Operativo:* Se detectó que el formato **MPEG (MP3)** domina los ingresos. Formatos pesados como AAC o Lossless consumen almacenamiento (costo de infraestructura) pero generan ingresos marginales. *Recomendación: Depurar catálogo.*

### SECCIÓN 4: Gestión de Talento y Estrategia de Producto

**Objetivo:** Evaluar el rendimiento del capital humano y la alineación del producto con el mercado.

- **Gráfico: Ranking de Performance de Vendedores**

- *Métricas:* Ingresos generados vs. Transacciones cerradas.

- *Insight:* Identificación de Top Performers para programas de mentoría interna y detección de necesidades de capacitación en el resto del equipo.

- **Gráfico: Top 10 Géneros Rentables (Cash Cows)**

- *Pregunta que responde:* ¿Qué contenido realmente paga las cuentas?

- *Insight:* Validación de tendencias de mercado (ej. predominancia de Rock/Latin sobre nichos como Opera o Drama). Esto guía la adquisición de nuevas licencias musicales.

---

## 5. Infraestructura y Despliegue (DevOps)

El proyecto simula un entorno de producción real, asegurando escalabilidad y seguridad.

- **Cloud Provider:** VPS en Clouding.io (Ubuntu 24.04 LTS).

- **Contenerización:** Docker & Docker Compose para orquestar los servicios.

- **Stack:**

- **Base de Datos:** PostgreSQL 16 (Persistencia de datos).

- **Gestión:** pgAdmin 4 (Administración y Backups).

- **Visualización:** Metabase (Business Intelligence Frontend).

- **Seguridad:** Configuración de Firewall perimetral permitiendo tráfico únicamente en puertos estrictamente necesarios (SSH/HTTP).

## 6. Conclusión

Este proyecto demuestra cómo una arquitectura de datos moderna y un análisis visual bien diseñado pueden transformar una base de datos estática en un centro de comando estratégico, permitiendo a la organización tomar decisiones basadas en evidencia (Data-Driven) en lugar de intuición.