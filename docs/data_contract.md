# Data Contract — Stellar Population Pipeline
> Versión 1.0 · Última actualización: 2026-04-02

## Fuente
- **Dataset:** NASA Exoplanet Archive — PSCompPars  
- **Acceso:** TAP API `https://exoplanetarchive.ipac.caltech.edu/TAP/sync`  
- **Enfoque científico:** Análisis de población estelar y Diagrama de Hertzsprung-Russell

---

## Datasets del pipeline

| Dataset | Etapa | Formato | Ubicación |
|---|---|---|---|
| `pscomppars_stellar_*.csv` | Raw | CSV | `data/raw/` |
| `bronze_stellar.parquet` | Bronze | Parquet/Snappy | `data/bronze/` |
| `silver_planet.parquet` | Silver | Parquet/Snappy | `data/silver/` |
| `dim_host_sk` | Gold/Dim | Parquet + CSV | `data/gold/` + `artifacts/` |
| `fact_planet_sk` | Gold/Fact | Parquet + CSV | `data/gold/` + `artifacts/` |
| `gold_by_spectral_class` | Gold | CSV | `artifacts/` |
| `gold_hr_diagram` | Gold | CSV | `artifacts/` |

---

## Grain

| Tabla | Grain |
|---|---|
| `dim_host_sk` | 1 fila por `hostname` (estrella anfitriona) |
| `fact_planet_sk` | 1 fila por `pl_name` (planeta) |
| `gold_by_spectral_class` | 1 fila por clase espectral (O, B, A, F, G, K, M) |
| `gold_hr_diagram` | 1 fila por `host_id` con coordenadas HR válidas |

---

## Keys

| Tabla | Clave | Tipo | Constraint |
|---|---|---|---|
| `dim_host_sk` | `host_id` | Surrogate (INT) | PRIMARY KEY, auto-incremental |
| `dim_host_sk` | `hostname` | Natural key (STR) | UNIQUE, NOT NULL |
| `fact_planet_sk` | `pl_name` | Natural key (STR) | PRIMARY KEY |
| `fact_planet_sk` | `host_id` | Foreign key (INT) | FK → `dim_host_sk.host_id` |

**Relación:** `fact_planet_sk.host_id → dim_host_sk.host_id`

---

## Columnas principales

### dim_host_sk
| Columna | Tipo | Descripción |
|---|---|---|
| `host_id` | INT | Surrogate key |
| `hostname` | STR | Nombre de la estrella anfitriona |
| `spectral_class` | STR | Clase espectral principal (O/B/A/F/G/K/M/Unknown) |
| `hr_region` | STR | Región en el diagrama HR |
| `st_teff_k` | FLOAT | Temperatura efectiva (K) |
| `st_mass_msun` | FLOAT | Masa estelar (M☉) |
| `st_rad_rsun` | FLOAT | Radio estelar (R☉) |
| `st_lum_log` | FLOAT | Luminosidad log(L/L☉) |
| `n_planets` | INT | Número de planetas en el sistema |

### fact_planet_sk
| Columna | Tipo | Descripción |
|---|---|---|
| `pl_name` | STR | Nombre del planeta (PK) |
| `host_id` | INT | FK → dim_host_sk |
| `pl_rad_rearth` | FLOAT | Radio planetario (R⊕) |
| `pl_orbper_d` | FLOAT | Período orbital (días) |
| `pl_eqt_k` | FLOAT | Temperatura de equilibrio (K) |
| `pl_size_class` | STR | Rocky / Super-Earth / Sub-Neptune / Neptune-like / Giant |
| `hab_zone_flag` | INT | 1 si T_eq ∈ [200, 320] K |
| `disc_year` | INT | Año de descubrimiento |
| `disc_method` | STR | Método de detección |

---

## Checks mínimos (con evidencia)

| Check | Expresión SQL | Estado |
|---|---|---|
| Unicidad dim | `COUNT(*) == COUNT(DISTINCT hostname)` | ✓ verificado en 03 |
| Unicidad fact | `COUNT(*) == COUNT(DISTINCT pl_name)` | ✓ verificado en 03 |
| Orphans | `orphan_rows == 0` | ✓ verificado en 03 |
| FK integridad | `fact.host_id ⊆ dim.host_id` | ✓ verificado en 03 |
| Rangos físicos | `st_teff_k ∈ [2000, 50000]`, etc. | ✓ aplicado en 02 |
| n_rows dim == n_keys | `n_rows(dim) == n_keys(hostname)` | ✓ verificado en 03 |

---

## SLA de calidad Silver

| Columna | % nulos máximo permitido |
|---|---|
| `hostname` | 0% |
| `pl_name` | 0% |
| `st_teff_k` | 5% |
| `st_mass_msun` | 10% |
| `st_lum_log` | 15% |
| `pl_rad_rearth` | 20% |
