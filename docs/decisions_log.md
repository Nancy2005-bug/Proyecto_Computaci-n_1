# Decisions Log — Stellar Population Pipeline

> Registro de decisiones técnicas y científicas clave durante el desarrollo del pipeline.

---

## D-01: Elección de columnas estelares vs. orbitales
**Fecha:** 2026-04-02  
**Contexto:** El dataset PSCompPars tiene >200 columnas. El profesor enfocó su pipeline en métodos de detección y parámetros orbitales planetarios (`pl_orbper`, `pl_ecc`, etc.).  
**Decisión:** Seleccionar columnas de propiedades estelares (`st_teff`, `st_mass`, `st_rad`, `st_lum`, `st_logg`, `st_spectype`) para construir un pipeline de análisis de población estelar y diagrama HR.  
**Justificación:**  
- Pregunta científica diferente: ¿cómo se relacionan las propiedades de las estrellas con la arquitectura de sus sistemas planetarios?  
- El diagrama Hertzsprung-Russell es fundamental en astrofísica estelar  .
- Evita redundancia con el trabajo del profesor  
**Alternativas descartadas:**  
- Usar otra fuente (GAIA DR3): demasiado volumen para el alcance del curso  
- Solo planetas sin estrella: pierde la dimensión del modelo relacional dim/fact  
**Impacto:** Define el esquema completo del pipeline y las preguntas gold.

---

## D-02: Estrategia de nulos en columnas estelares
**Fecha:** 2026-04-02  
**Contexto:** Columnas como `st_teff_k` y `st_mass_msun` tienen entre 5-15% de nulos. Simplemente eliminar filas perdería sistemas completos.  
**Decisión:** Imputar con mediana agrupada por `disc_method` (método de descubrimiento).  
**Justificación:**  
- Planetas descubiertos con el mismo método tienden a orbitar estrellas similares (sesgo de selección observacional).  
- La mediana es robusta a outliers, importante para distribuciones asimétricas de masas estelares.  
- Se documenta como transformación reversible (dato original en Bronze intacto).  
**Alternativas descartadas:**  
- Imputación por mediana global: ignora el sesgo de selección observacional.  
- KNN imputation: añade complejidad computacional innecesaria para el scope del curso.  
- Eliminación de filas: pérdida de ~15% de sistemas únicos  
**Impacto:** ~800-1200 valores imputados en columnas clave; se documenta en Silver.

---

## D-03: Definición de `hr_region` con `st_logg`
**Fecha:** 2026-04-02  
**Contexto:** Clasificar estrellas en el diagrama HR requiere una métrica de luminosidad de clase. `st_lum_log` tiene más nulos que `st_logg`.  
**Decisión:** Usar `st_logg` (log g de gravedad superficial) como proxy principal para clasificar entre Secuencia Principal, Subgigantes, Gigantes y Supergigantes.  
**Justificación:**  
- `log g > 4.0` → Secuencia principal; `2.5 < log g ≤ 3.5` → Gigante; `log g ≤ 2.5` → Supergigante.  
- Mayor completitud que `st_lum_log` en el catálogo.  
- Clasificación consistente con literatura estelar estándar (Gray & Corbally 2009).  
**Alternativas descartadas:**  
- Solo `st_teff + st_lum_log`: mayor potencia física pero ~15% más nulos.  
**Impacto:** Permite clasificar >90% de las estrellas sin datos faltantes críticos.

---

## D-04: Formato Parquet con compresión Snappy para capas internas
**Fecha:** 2026-04-02  
**Contexto:** Elegir formato de almacenamiento para Bronze y Silver.  
**Decisión:** Parquet con compresión Snappy para todas las capas internas (Bronze, Silver, Gold/dim, Gold/fact).  
**Justificación (Pilar: Scalability):**  
- Parquet es columnar → lecturas parciales eficientes con DuckDB.  
- Snappy: balance óptimo entre ratio de compresión y velocidad de decompresión.  
- Reducción de tamaño ~60-70% vs CSV.  
- DuckDB puede consultar Parquet directamente sin cargar en memoria.  
**Artefactos de evidencia:** CSV en `artifacts/` para trazabilidad humana; Parquet en `data/` para el pipeline.  
**Alternativas descartadas:**  
- CSV puro: sin tipado, lento para lecturas parciales.  
- Parquet + Zstd: mejor compresión pero más lento para acceso aleatorio.