# 📊 Chinook Strategy Command Center: Solución de Business Intelligence End-to-End

## 📑 Tabla de Contenidos
- [Resumen](#-resumen-ejecutivo)
- [Contexto de negocio y problemática](#-contexto-de-negocio-y-problemática)
- [Marco estratégico: OKRs y KPIs](#-marco-estratégico-okrs-y-kpis)
- [Arquitectura de datos e ingeniería](#️-arquitectura-de-datos-e-ingeniería)
- [Modelado dimensional (OLAP)](#-modelado-dimensional-olap)
- [Análisis profundo y segmentación de clientes (RFM)](#-análisis-profundo-y-segmentación-de-clientes-rfm)
- [Descripción del dashboard e insights](#-descripción-del-dashboard-e-insights)
- [Stack tecnológico](#-stack-tecnológico)
- [Instalación y despliegue](#-instalación-y-despliegue)

## 🚀 Resumen 
Este proyecto presenta una solución integral de Business Intelligence (BI) diseñada para transformar los datos transaccionales de una tienda de medios digitales (Chinook) en activos estratégicos para la toma de decisiones.

Simulando un entorno corporativo real, se realizó una migración completa de una arquitectura **OLTP** (On-Line Transaction Processing) hacia un **Data Warehouse OLAP** (On-Line Analytical Processing). El resultado es un sistema capaz de procesar volúmenes de datos históricos, reducir la latencia de consulta y visualizar métricas críticas de negocio mediante un dashboard interactivo autogestionado.

## 🏢 Contexto de negocio y problemática
**Chinook** es una tienda global de música y video digital. A pesar de contar con una base de datos robusta, la organización sufría de "Ceguera Operativa":

- **Silos de datos**: La información estaba dispersa en tablas altamente normalizadas, dificultando la visión holística.
- **Latencia en reportes**: Consultas analíticas complejas sobre el sistema transaccional degradaban el rendimiento de la operación diaria.
- **Falta de segmentación**: No existía una metodología para identificar clientes de alto valor o riesgos de abandono (Churn).

**La solución**: Implementación de un pipeline ETL (Extract, Transform, Load) para consolidar una "Single Source of Truth" (SSOT) en un esquema dimensional optimizado para lectura.

## 🎯 Marco estratégico: OKRs y KPIs
El diseño del dashboard responde a objetivos estratégicos definidos bajo la metodología **OKR** (Objectives and Key Results):

### Objetivo 1: Maximizar la eficiencia de ingresos
**KR 1**: Monitorear el flujo de caja histórico y actual para identificar estacionalidades.
- **KPI principal**: Total Revenue (Ingresos Brutos).
- **KPI secundario**: AOV (Average Order Value) - Ticket promedio por transacción.

### Objetivo 2: Optimización de inventario y operaciones
**KR 1**: Identificar los formatos de archivo más rentables vs. los que consumen almacenamiento innecesario.
- **KPI**: Profitability by Media Type.

### Objetivo 3: Retención y fidelización de clientes
**KR 1**: Segmentar la base de usuarios basada en comportamiento de compra para personalizar estrategias de marketing.
- **KPI**: Customer Lifetime Value (CLV) proxies mediante segmentación RFM.

## ⚙️ Arquitectura de datos e ingeniería
El flujo de trabajo sigue un enfoque de ingeniería de datos moderno, orquestado mediante contenedores para asegurar la reproducibilidad.

### Flujo del pipeline (ETL)
1. **Extracción (Source)**: Se parte de una base de datos PostgreSQL en 3FN (Tercera Forma Normal). Este modelo relacional es eficiente para la integridad de datos (escritura) pero ineficiente para el análisis (lectura) debido a la excesiva cantidad de JOINs necesarios.

2. **Transformación (Staging Area)**: Mediante scripts SQL avanzados (DDL/DML), se limpian los datos, se desnormalizan tablas y se calculan métricas pre-agregadas.

3. **Carga (Target)**: Los datos transformados se insertan en un esquema dimensional (Star Schema) dentro del Data Warehouse.

### Infraestructura (IaaS)
El proyecto se despliega sobre un servidor VPS Linux (Ubuntu 24.04) en la nube, utilizando **Docker Compose** para levantar una arquitectura multi-contenedor que incluye la base de datos y la plataforma de visualización (Metabase), interconectados en una red interna aislada.

## 📐 Modelado dimensional (OLAP)
Para optimizar el rendimiento de las consultas analíticas, se diseñó un **Esquema de Estrella (Star Schema)**:

![Modelo OLTP](/assets/oltp_model.png)

*Modelo OLTP*

![Modelo OLAP (Esquema estrella)](/assets/olap_model.png)

*Modelo OLAP (Esquema estrella)*

- **Fact Table (fact_invoice_lines)**: Tabla central de hechos que contiene las métricas cuantitativas (precios unitarios, cantidades, totales) y las llaves foráneas.

- **Dimension tables**: Tablas satelitales desnormalizadas que aportan contexto:
  - `dim_customers`: Datos demográficos y ubicación.
  - `dim_tracks`: Metadatos de las canciones, álbumes, géneros y tipos de medio.
  - `dim_time`: (Implícita) Para cortes temporales y análisis de series de tiempo.
  - `dim_employees`: Jerarquía organizacional y ventas.

Este modelo reduce la complejidad de las consultas de **6+ JOINs** (en OLTP) a **1 o 2 uniones simples**, acelerando drásticamente el tiempo de respuesta del dashboard.

## 💎 Análisis profundo y segmentación de clientes (RFM)
La "Joya de la Corona" de este análisis es la implementación de un algoritmo de **Segmentación RFM** (Recencia, Frecuencia, Valor Monetario) utilizando SQL Avanzado.

### Metodología técnica
A diferencia de herramientas "caja negra", aquí se calculó la segmentación manualmente en la base de datos utilizando:

- **CTEs (Common Table Expressions)**: Para aislar métricas por cliente.
- **Window Functions (NTILE)**: Para dividir a la población en cuartiles estadísticos (scores del 1 al 4) en tres dimensiones:
  - **Recencia (R)**: ¿Hace cuánto compró? (1 = Lejano, 4 = Reciente).
  - **Frecuencia (F)**: ¿Cuántas veces compró?
  - **Monetario (M)**: ¿Cuánto gastó en total?

### Visualización e insights
Se construyó un **Scatter Plot** (Matriz de Dispersión) correlacionando el Valor Monetario (Eje Y) vs. Frecuencia/Recencia (Eje X).

**Objetivo**: Identificar clusters de comportamiento para acciones tácticas.

**Insight clave**:
- **Campeones (Superior derecha)**: Clientes con alta frecuencia y alto gasto. **Acción**: Programas de fidelidad VIP.
- **En Riesgo (Superior izquierda)**: Clientes que gastaron mucho en el pasado pero no han vuelto (Baja Recencia). **Acción**: Campañas de reactivación/Win-back agresivas.
- **Nuevos/Prometedores**: Clientes recientes con potencial de crecimiento.

## 📊 Descripción del dashboard e insights
El tablero de control en **Metabase** se estructura en niveles de lectura:

1. **Filtros globales**: Permiten al usuario segmentar todo el reporte por Rango de Fechas y País de Facturación (Billing Country), otorgando interactividad total.

2. **Health check (KPIs)**: Tarjetas numéricas con indicadores de Ingresos Totales y Promedio de Venta, permitiendo una evaluación instantánea del estado financiero.

3. **Análisis geoespacial**: Mapa de calor que muestra la densidad de ventas por país.
   - **Insight**: América del Norte y Europa concentran el 80% del mercado, sugiriendo focalizar esfuerzos logísticos y de marketing en estas zonas.

4. **Análisis de pareto (Formatos)**: Gráfico de barras que contrasta ingresos por tipo de archivo.
   - **Insight**: A pesar de almacenar archivos pesados (AAC/Lossless), el formato MPEG (MP3) genera la inmensa mayoría de los ingresos. Esto sugiere una oportunidad de ahorro en costos de almacenamiento cloud eliminando formatos de baja rotación.

5. **Performance de empleados**: Ranking de ventas por agente de soporte, útil para evaluaciones de desempeño y bonificaciones.

## 🛠 Stack tecnológico
- **Base de datos**: PostgreSQL 16 (Motor relacional robusto).
- **Lenguajes**: SQL (PL/pgSQL para procedimientos almacenados), Bash (Scripting).
- **Infraestructura**: Docker & Docker Compose (Contenerización).
- **Cloud**: Ubuntu Server en VPS (Clouding.io).
- **Visualización**: Metabase (Business Intelligence Open Source).
- **Control de versiones**: Git & GitHub.

## 💻 Instalación y despliegue
Sigue estos pasos para desplegar el proyecto en tu entorno local:

1. **Clonar el repositorio**:
   ```bash
   git clone https://github.com/tu-usuario/chinook-strategy-center.git
   cd chinook-strategy-center
   ```

2. **Configurar variables de entorno**: Renombrar el archivo `.env.example` a `.env` y configurar las credenciales de base de datos.

3. **Despliegue con Docker**:
   ```bash
   docker-compose up -d --build
   ```

4. **Ejecución del ETL**: Acceder al contenedor de base de datos y ejecutar el script de migración SQL provisto en `/scripts/etl_pipeline.sql`.

5. **Acceso**:
   - Metabase: http://localhost:3000
   - Base de datos: Puerto 5432

---

**Este proyecto fue desarrollado como trabajo final para el curso de Data Analytics en Devlights.**