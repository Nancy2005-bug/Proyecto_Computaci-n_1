# 🌟 Stellar Population Pipeline
### Física Computacional 1 — Ingeniería de Datos | Proyecto W05

Pipeline reproducible de ingeniería de datos sobre el catálogo NASA Exoplanet Archive (PSCompPars),
enfocado en **análisis de población estelar y Diagrama de Hertzsprung-Russell**.

> **Pregunta científica:**  
> *¿Cómo varía la arquitectura de los sistemas planetarios (tamaño, período orbital, zona habitable)  
> según la clase espectral de la estrella anfitriona?*

---

## 🗂️ Estructura del repositorio

```
stellar-pipeline/
├── notebooks/
│   ├── 00_raw_ingestion.ipynb      # Descarga + SHA-256 + metadata
│   ├── 01_bronze_lite.ipynb        # Tipado canónico + Parquet
│   ├── 02_silver_cleaning.ipynb    # Limpieza + spectral_class + hr_region
│   ├── 03_dim_fact_modeling.ipynb  # dim_host_sk + fact_planet_sk + FK checks
│   └── 04_gold_analysis.ipynb      # Gold outputs + Diagrama HR
├── data/
│   ├── raw/           # CSV original + metadata.json + SHA-256 (NO versionar)
│   ├── bronze/        # Parquet tipado
│   ├── silver/        # Parquet limpio + enriquecido
│   └── gold/          # dim_host_sk.parquet + fact_planet_sk.parquet
├── artifacts/         # Evidencias CSV + PNG (versionar)
│   ├── dim_host_sk.csv
│   ├── fact_planet_sk.csv
│   ├── gold_by_spectral_class.csv
│   ├── gold_hr_diagram.csv
│   └── hr_diagram_analysis.png
├── docs/
│   ├── data_contract.md    # Grains, keys, checks, SLA
│   └── decisions_log.md    # 4 decisiones técnicas documentadas
├── requirements.txt
├── .gitignore
└── README.md
```

---

## ⚡ Quickstart (VS Code)

```bash
# 1. Clonar y entrar al repositorio
git clone <tu-repo> && cd stellar-pipeline

# 2. Crear entorno virtual
python -m venv .venv
# Linux/Mac:
source .venv/bin/activate
# Windows PowerShell:
.venv\Scripts\Activate.ps1

# 3. Instalar dependencias
pip install -r requirements.txt

# 4. Abrir VS Code
code .

# 5. Ejecutar notebooks en orden (Ctrl+Shift+P → "Select Kernel" → .venv)
# notebooks/00_raw_ingestion.ipynb  → descarga datos
# notebooks/01_bronze_lite.ipynb    → Bronze
# notebooks/02_silver_cleaning.ipynb → Silver
# notebooks/03_dim_fact_modeling.ipynb → Dim/Fact
# notebooks/04_gold_analysis.ipynb  → Gold + visualizaciones
```

---

## 🔬 Pipeline: Raw → Bronze → Silver → Gold

```
NASA TAP API
    │
    ▼
[RAW] pscomppars_stellar_YYYYMMDD.csv
  + SHA-256 hash + metadata.json
    │
    ▼ 01_bronze_lite.ipynb
[BRONZE] bronze_stellar.parquet (Snappy)
  Tipado canónico, sin transformaciones de negocio
    │
    ▼ 02_silver_cleaning.ipynb
[SILVER] silver_planet.parquet
  + filtros físicos + spectral_class + hr_region + pl_size_class
    │
    ├──► dim_host_sk  (1 fila/estrella, host_id PK)
    │
    └──► fact_planet_sk (1 fila/planeta, host_id FK)
              │
              ├──► gold_by_spectral_class.csv
              └──► gold_hr_diagram.csv + hr_diagram_analysis.png
```

---

## 🏛️ Tres Pilares

| Pilar | Implementación |
|---|---|
| **Reliability** | SHA-256 en raw, checks PK/FK/orphans en notebook 03, filtros de rangos físicos |
| **Scalability** | Parquet + Snappy columnar, DuckDB SQL directo sobre Parquet sin cargar en RAM |
| **Maintainability** | `data_contract.md` con grains/keys/checks, `decisions_log.md` con 4 decisiones, notebooks numerados y secuenciales |

---

## 📊 Columnas seleccionadas (enfoque estelar — distinto al del profesor)

| Grupo | Columnas |
|---|---|
| **Estrella (HR Diagram)** | `st_teff`, `st_rad`, `st_mass`, `st_lum`, `st_logg`, `st_met`, `st_spectype` |
| **Sistema** | `hostname`, `sy_snum`, `sy_pnum`, `sy_dist` |
| **Planeta** | `pl_name`, `pl_orbper`, `pl_rade`, `pl_bmasse`, `pl_eqt` |
| **Descubrimiento** | `disc_year`, `discoverymethod` |

---

## 📋 Data Contract (resumen)

- `dim_host_sk`: 1 fila por `hostname` | PK: `host_id` (surrogate), UNIQUE: `hostname`
- `fact_planet_sk`: 1 fila por `pl_name` | PK: `pl_name`, FK: `host_id → dim_host_sk.host_id`
- Checks: `orphan_rows == 0`, `n_rows(dim) == n_keys(hostname)`, unicidad PK en ambas tablas

Ver [`docs/data_contract.md`](docs/data_contract.md) y [`docs/decisions_log.md`](docs/decisions_log.md)

---

## 🛠️ Stack tecnológico

| Herramienta | Versión | Uso |
|---|---|---|
| Python | 3.11 | Lenguaje principal |
| DuckDB | 1.4.4 | SQL analítico sobre Parquet |
| Pandas | 2.3.3 | Transformaciones en memoria |
| PyArrow | 21.0.0 | Lectura/escritura Parquet |
| Matplotlib | latest | Visualizaciones HR |
| Requests | 2.32.5 | Descarga NASA TAP API |

---

## 👤 Autor
Curso: Física Computacional 1 — Ingeniería de Datos Fundamentos  
Docente: Ph.D. Santiago Echeverri Arteaga
