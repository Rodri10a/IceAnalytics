# ICE Analytics - Product Activity Analysis

## Descripcion General

Proyecto de analisis de datos enfocado en evaluar la actividad de producto de una plataforma donde los usuarios crean posts y reciben votos. El analisis aborda un problema real: **los datos en produccion no son confiables tal como llegan**. Se detectan inconsistencias, se limpian con criterio documentado y se extraen metricas accionables para decisiones de producto.

---

## Problema

El tablero de producto no cerraba. Las metricas mostraban numeros inflados y segmentaciones incorrectas debido a:

- Variantes sucias en campos categoricos (`Free`, `FREE`, `FreE`, `free` como entradas distintas)
- 51% de inconsistencias en la columna `days_since_signup`
- Duplicados exactos inflando conteos de actividad
- Posts con fecha anterior al signup del usuario

Este proyecto documenta el proceso completo de deteccion, medicion y correccion de estos problemas.

---

## Estructura del Proyecto

```
IceAnalytics/
├── ice.ipynb                          # Notebook principal con todo el analisis
├── product_activity.csv               # Dataset original (raw)
├── clean_product_activity.csv         # Dataset limpio (output)
├── quarantine_product_activity.csv    # Filas excluidas con motivo (output)
├── metrics_summary.csv               # Metricas resumen (output)
├── requirements.txt                   # Dependencias del proyecto
├── .gitignore
└── README.md
```

---

## Dataset

**Fuente:** `product_activity.csv` — 8,782 filas x 12 columnas

**Unidad de analisis:** Cada fila es un **evento (post)**, no un usuario. Un usuario con 50 posts aparece en 50 filas. Esto tiene implicaciones directas en como se calculan promedios y se interpretan metricas.

| Columna | Tipo | Descripcion |
|---------|------|-------------|
| `user_id` | string | Identificador unico del usuario |
| `created_at` | datetime | Fecha de registro (signup) |
| `country` | string | Pais del usuario |
| `plan_type` | string | Plan de suscripcion (free / pro / enterprise) |
| `user_age` | float | Edad del usuario (754 valores nulos) |
| `post_id` | string | Identificador unico del post |
| `post_category` | string | Categoria del contenido |
| `post_created_at` | datetime | Fecha de creacion del post |
| `votes_received` | int | Votos recibidos por el post |
| `user_total_posts` | int | Total historico de posts del usuario |
| `days_since_signup` | int | Dias entre signup y post (no confiable) |
| `device_type` | string | Dispositivo utilizado |

---

## Metodologia

El notebook sigue un flujo estructurado en 7 etapas:

### 1. Exploracion Inicial
Medicion del estado de los datos antes de cualquier transformacion: distribucion de tipos, nulos, duplicados, valores unicos por columna y chequeos logicos de consistencia temporal.

### 2. Limpieza
- **Deduplicacion:** Remocion de 172 filas duplicadas exactas
- **Normalizacion canonica:** Mapeo de variantes sucias a diccionarios fijos para `plan_type`, `post_category` y `device_type`
- **Recalculo de fechas:** Generacion de `days_since_signup_calc` a partir de las fechas reales
- **Quarantine:** Separacion de 102 filas con errores irrecuperables (posts antes del signup, fechas no parseables, valores fuera de diccionario), cada una con su `reason_code`

### 3. Data Quality Report
Resumen cuantitativo del impacto de la limpieza sobre el dataset.

### 4. Metricas y Segmentacion
- Distribuciones de volumen por plan, pais, categoria y dispositivo
- Engagement (votos) segmentado con media, mediana y percentiles
- Analisis de sesgo: comparacion de metricas a nivel evento vs nivel usuario

### 5. Concentracion y Temporalidad
- Participacion del top 1% de usuarios en posts y votos
- Tendencias mensuales de actividad y engagement

### 6. Product Decisions
Conclusiones basadas en evidencia: segmento a priorizar, que parte del tablero reportaba datos incorrectos, y dos acciones concretas con respaldo cuantitativo y limitaciones explicitas.

### 7. Exportacion
Generacion de tres archivos CSV listos para consumo por otros equipos.

---

## Resultados Clave

| Metrica | Valor |
|---------|-------|
| Filas originales | 8,782 |
| Duplicados removidos | 172 |
| Filas en quarantine | 102 (1.2%) |
| Filas limpias (CORE) | 8,508 |
| Usuarios unicos | 1,995 |
| Promedio de votos por evento | 6.91 |
| Mediana de votos | 6.0 |
| Mismatches en `days_since_signup` | 51.1% |
| Concentracion top 1% | 6.0% de posts y votos |

---

## Archivos de Salida

| Archivo | Filas | Descripcion |
|---------|-------|-------------|
| `clean_product_activity.csv` | 8,508 | Dataset limpio con columnas normalizadas |
| `quarantine_product_activity.csv` | 102 | Filas excluidas con columna `reason_code` |
| `metrics_summary.csv` | 10 | Tabla de metricas principales del analisis |

---

## Requisitos e Instalacion

**Python 3.10+**

```bash
pip install -r requirements.txt
```

### Ejecucion

```bash
jupyter notebook ice.ipynb
```

Ejecutar todas las celdas en orden. Los archivos CSV se generan automaticamente en la ultima celda.

---

## Tecnologias

- **pandas** — Manipulacion y limpieza de datos
- **numpy** — Operaciones numericas
- **matplotlib** — Visualizacion base
- **seaborn** — Visualizacion estadistica
- **Jupyter Notebook** — Entorno de analisis interactivo
