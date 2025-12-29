# 📊 Chinook Strategy Command Center: Solución de Business Intelligence End-to-End

![Status](https://img.shields.io/badge/Status-Completed-success) ![PostgreSQL](https://img.shields.io/badge/PostgreSQL-16-blue) ![Docker](https://img.shields.io/badge/Docker-Compose-2496ED) ![Metabase](https://img.shields.io/badge/Visualization-Metabase-509EE3)

## 📑 Tabla de Contenidos
- [Resumen](#-resumen-ejecutivo)
- [Contexto de negocio y problemática](#-contexto-de-negocio-y-problemática)
- [Marco estratégico: OKRs y KPIs](#-marco-estratégico-okrs-y-kpis)
- [Arquitectura de datos e ingeniería](#-arquitectura-de-datos-e-ingeniería)
- [Modelado dimensional (OLAP)](#-modelado-dimensional-olap)
- [Análisis profundo y segmentación (RFM)](#-análisis-profundo-y-segmentación-de-clientes-rfm)
- [Descripción del dashboard e insights](#-descripción-del-dashboard-e-insights)
- [Stack tecnológico](#-stack-tecnológico)
- [Instalación y despliegue](#-instalación-y-despliegue)

## 🚀 Resumen 
Este proyecto presenta una solución integral de **Business Intelligence (BI)** diseñada para transformar los datos transaccionales de una tienda de medios digitales (*Chinook*) en activos estratégicos para la toma de decisiones.

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
- **KPI Principal**: *Total Revenue* (Ingresos Brutos).
- **KPI Secundario**: *AOV (Average Order Value)* - Ticket promedio por transacción.

### Objetivo 2: Optimización de inventario y operaciones
**KR 1**: Identificar los formatos de archivo más rentables vs. los que consumen almacenamiento innecesario.
- **KPI**: *Profitability by Media Type*.

### Objetivo 3: Retención y fidelización de clientes
**KR 1**: Segmentar la base de usuarios basada en comportamiento de compra para personalizar estrategias de marketing.
- **KPI**: *Customer Lifetime Value (CLV)* proxies mediante segmentación RFM.

## ⚙️ Arquitectura de datos e ingeniería
El flujo de trabajo sigue un enfoque de ingeniería de datos moderno, orquestado mediante contenedores para asegurar la reproducibilidad.

### Flujo del Pipeline (ETL)
1. **Extracción (Source)**: Se parte de una base de datos PostgreSQL en **3FN (Tercera Forma Normal)**. Este modelo relacional es eficiente para la integridad de datos (escritura) pero ineficiente para el análisis (lectura) debido a la excesiva cantidad de `JOINs` necesarios.

2. **Transformación (Staging Area)**: Mediante scripts SQL avanzados (DDL/DML), se limpian los datos, se desnormalizan tablas y se calculan métricas pre-agregadas.

3. **Carga (Target)**: Los datos transformados se insertan en un esquema dimensional (**Star Schema**) dentro del Data Warehouse.

### Infraestructura (IaaS)
El proyecto se despliega sobre un servidor **VPS Linux (Ubuntu 24.04)** en la nube, utilizando **Docker Compose** para levantar una arquitectura multi-contenedor que incluye la base de datos, la plataforma de visualización (Metabase) y herramientas de administración (pgAdmin), interconectados en una red interna aislada.

## 📐 Modelado dimensional (OLAP)
Para optimizar el rendimiento de las consultas analíticas, se diseñó un **Esquema de Estrella (Star Schema)**:

![Modelo OLTP](/assets/oltp_model.png)
*Fig 1. Modelo Relacional Original (OLTP)*

![Modelo OLAP (Esquema estrella)](/assets/olap_model.png)
*Fig 2. Modelo Dimensional Optimizado (OLAP)*

- **Fact Table (`fact_invoice_lines`)**: Tabla central de hechos que contiene las métricas cuantitativas (precios unitarios, cantidades, totales) y las llaves foráneas.
- **Dimension tables**: Tablas satelitales desnormalizadas que aportan contexto:
  - `dim_customers`: Datos demográficos y ubicación.
  - `dim_tracks`: Metadatos de las canciones, álbumes, géneros y tipos de medio.
  - `dim_time`: (Implícita) Para cortes temporales y análisis de series de tiempo.
  - `dim_employees`: Jerarquía organizacional y ventas.

Este modelo reduce la complejidad de las consultas de **6+ JOINs** (en OLTP) a **1 o 2 uniones simples**, acelerando drásticamente el tiempo de respuesta del dashboard.

## 💎 Análisis profundo y segmentación de Clientes (RFM)
La "Joya de la Corona" de este análisis es la implementación de un algoritmo de **Segmentación RFM** (Recencia, Frecuencia, Valor Monetario) utilizando SQL Avanzado.

### Metodología técnica
A diferencia de herramientas "caja negra", aquí se calculó la segmentación manualmente en la base de datos utilizando:

- **CTEs (Common Table Expressions)**: Para aislar métricas por cliente.
- **Window Functions (`NTILE`)**: Para dividir a la población en cuartiles estadísticos (scores del 1 al 4) en tres dimensiones:
  - **Recencia (R)**: ¿Hace cuánto compró? (1 = Lejano, 4 = Reciente).
  - **Frecuencia (F)**: ¿Cuántas veces compró?
  - **Monetario (M)**: ¿Cuánto gastó en total?

### Visualización e insights
Se construyó un **Scatter Plot** (Matriz de Dispersión) correlacionando el *Valor Monetario* (Eje Y) vs. *Frecuencia/Recencia* (Eje X).

**Objetivo**: Identificar clusters de comportamiento para acciones tácticas.

**Insight Clave**:
- **Campeones (Superior Derecha)**: Clientes con alta frecuencia y alto gasto. **Acción**: Programas de fidelidad VIP.
- **En Riesgo (Superior Izquierda)**: Clientes que gastaron mucho en el pasado pero no han vuelto (Baja Recencia). **Acción**: Campañas de reactivación/Win-back agresivas.
- **Nuevos/Prometedores**: Clientes recientes con potencial de crecimiento.

## 📊 Descripción del Dashboard e Insights
El tablero de control en **Metabase** se estructura en niveles de lectura:

1. **Filtros Globales**: Permiten al usuario segmentar todo el reporte por *Rango de Fechas* y *País de Facturación* (Billing Country), otorgando interactividad total.

2. **Health Check (KPIs)**: Tarjetas numéricas con indicadores de *Ingresos Totales* y *Promedio de Venta*, permitiendo una evaluación instantánea del estado financiero.

3. **Análisis Geoespacial**: Mapa de calor que muestra la densidad de ventas por país.
   - *Insight*: América del Norte y Europa concentran el 80% del mercado, sugiriendo focalizar esfuerzos logísticos y de marketing en estas zonas.

4. **Análisis de Pareto (Formatos)**: Gráfico de barras que contrasta ingresos por tipo de archivo.
   - *Insight*: A pesar de almacenar archivos pesados (AAC/Lossless), el formato **MPEG (MP3)** genera la inmensa mayoría de los ingresos. Esto sugiere una oportunidad de ahorro en costos de almacenamiento cloud eliminando formatos de baja rotación.

5. **Performance de Empleados**: Ranking de ventas por agente de soporte, útil para evaluaciones de desempeño y bonificaciones.

## 🛠 Stack Tecnológico
- **Base de Datos**: PostgreSQL 16 (Motor relacional robusto).
- **Lenguajes**: SQL (PL/pgSQL para procedimientos almacenados), Bash (Scripting).
- **Infraestructura**: Docker & Docker Compose (Contenerización).
- **Cloud**: Ubuntu Server en VPS (Clouding.io).
- **Visualización**: Metabase (Business Intelligence Open Source).
- **Control de Versiones**: Git & GitHub.

## 💻 Instalación y Despliegue

Este proyecto está completamente contenerizado ("Dockerized"). Gracias a la orquestación con **Docker Compose**, el despliegue del Data Warehouse, la ejecución del pipeline ETL y la configuración de la herramienta de visualización ocurren automáticamente con un solo comando.

### Requisitos Previos
- Docker & Docker Compose instalados.

### Pasos para Ejecutar:

1. **Clonar el repositorio**:
   ```bash
   git clone https://github.com/tu-usuario/chinook-strategy-center.git
   cd chinook-strategy-center
   ```

2. **Iniciar el entorno**:
   
   Ejecutar el siguiente comando en la raíz del proyecto. Esto descargará las imágenes, levantará los servicios y **ejecutará automáticamente los scripts de migración de datos (OLTP -> OLAP)**.
   
   ```bash
   docker-compose up -d
   ```
   
   *(Nota: El primer inicio puede demorar unos segundos mientras PostgreSQL procesa los scripts de inicialización).*

3. **Acceder a los Servicios**:

   **A. Dashboard de Negocio (Metabase)**
   
   *El entorno levanta una instancia con tableros pre-cargados.*
   
   - **URL**: http://localhost:3000
   - **Usuario**: `admin@chinook.com`
   - **Contraseña**: `Devlights2024`

   **B. Administración de Base de Datos (pgAdmin 4)**
   
   - **URL**: http://localhost:5050
   - **Usuario**: `admin@chinook.com`
   - **Contraseña**: `root`
   - **Configuración de conexión al servidor**:
     - Host name/address: `db`
     - Username: `devlights_user`
     - Password: `devlights_password`

4. **Detener el entorno**:
   ```bash
   docker-compose down
   ```
