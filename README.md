# Pricing Actuarial — Responsabilidad Civil

Proyecto de modelado actuarial para una cartera de seguros de **Responsabilidad Civil (RC)**, desarrollado como práctica del curso de Pricing. Se implementan técnicas de simulación, análisis exploratorio y **Modelos Lineales Generalizados (GLMs)** para estimar primas puras de forma técnica y reproducible.

---

##  Integrantes

| Nombre |
|--------|
| Cortés Sanginez Jorge Orlando |
| Gascón Calixto Ruben |
| Gonzalez Díaz Garzón Gerardo Felipe |
| Hernández Mejía Geovanna |
| Salmoran Acuña Jonathan Ivan |

---

##  Estructura del repositorio

```
Pricing/
├── Parte 1.ipynb              # Simulación de cartera de RC
├── Parte 2.ipynb              # Unión de bases y análisis de sobredispersión
├── Parte 3.ipynb              # GLMs: frecuencia, severidad y prima pura
└── data/
    ├── vigor.csv              # Pólizas en vigor (50,000 registros)
    ├── siniestros.csv         # Siniestros reportados (7,569 registros)
    ├── BaseTrabajo.csv        # Base unificada vigor + siniestros
    └── base_trabajo.pkl       # Base de trabajo serializada (pickle)
```

---

##  Descripción de los notebooks

### Parte 1 — Simulación de Cartera de Responsabilidad Civil

Construcción de un dataset sintético de **~200,000 pólizas de RC** con parámetros calibrados a partir de datos reales de **SESA 2024**. Las coberturas simuladas son:

- **AeI** — Actividades e Instalaciones
- **DO** — Directors & Officers
- **EO** — Errors & Omissions
- **RChosp** — RC Hospitales
- **RCprof** — RC Profesional

Se simulan frecuencias y severidades por segmento, se calculan **primas puras** por cobertura y se evalúa el **Loss Ratio (LR)** de la cartera resultante.

---

### Parte 2 — Unión de bases y selección de distribución

**Ejercicio 1 — Preparación de la base de trabajo**

Une la base de pólizas en vigor (`vigor.csv`) con la base de siniestros (`siniestros.csv`) mediante un *left join* por número de póliza (`NUMPOL`), filtrando la cobertura `1RC`. La base resultante (`BaseTrabajo.csv`) contiene exposición, características del riesgo, monto de siniestro y conteo de siniestros por póliza.

Se abordan preguntas conceptuales sobre la importancia de las llaves de unión y el impacto de llaves faltantes en el cálculo de la frecuencia.

**Ejercicio 5 — Sobredispersión y elección de distribución (AAR)**

Se evalúa la sobredispersión mediante los cocientes Pearson/df y Deviance/df, y se comparan cuatro modelos para la frecuencia:

| Modelo | Descripción |
|--------|-------------|
| Poisson | Línea base para conteos |
| Quasi-Poisson | Ajuste por sobredispersión |
| Zero-Inflated | Para exceso de ceros |
| Binomial Negativa | Para sobredispersión estructural |

---

### Parte 3 — GLMs en Pricing

Notebook principal con el ciclo completo de modelado actuarial para una cartera simulada de RC.

**Pregunta 1 — Simulación de datos de RC**

Se generan 10,000 registros con variables como `Giro` (Administrativo, Médico, Industrial, Tecnológico), `Causa` (Accidental, Negligencia, Productos) y `Tipo_Seguro` (RC General, RC Profesional).

**Pregunta 2 — Modelo de Frecuencia**

$$\text{Frequency}_i = \frac{\text{ClaimNb}_i}{\text{Exposure}_i}$$

Se ajustan modelos **Poisson** y **Binomial Negativa** con offset de exposición. El modelo Poisson resultó suficiente y parsimonioso (menor AIC), recuperando correctamente los coeficientes simulados:

- El giro **Médico** presenta ~2.5× la frecuencia del giro Administrativo (referencia)
- La causa **Negligencia** incrementa la frecuencia ~1.9×
- El seguro **RC Profesional** tiene una frecuencia ~1.16× mayor que RC General

**Pregunta 3 — Tasas de frecuencia por variable**

Visualización de tasas de frecuencia con **intervalos de confianza de Wald** para cada variable categórica, validando la coherencia de los parámetros simulados.

**Pregunta 4 — Modelo de Severidad**

$$\text{Severity}_i = \frac{\text{ClaimAmount}_i}{\max\{\text{ClaimNb}_i, 1\}}$$

Se ajusta un **GLM Gamma con liga log**, ponderado por número de siniestros. Se compara el modelo completo contra uno con siniestros recortados al **percentil 95** (menor AIC: 16,732 vs 17,893). La distribución **Weibull** se usa como contraste.

**Pregunta 5 — Prima Pura y evaluación del modelo**

$$\text{Prima Pura}_i = \hat{\text{Frecuencia}}_i \times \hat{\text{Severidad}}_i$$

Se calculan primas puras por póliza, se evalúa la discriminación con el **índice de Gini** y se obtienen predicciones parciales y factores multiplicativos respecto a una póliza de referencia.

---

## Descripción de los datos

### `vigor.csv` — Pólizas en vigor

| Campo | Descripción |
|-------|-------------|
| `NUMPOL` | Número de póliza (llave primaria) |
| `EXPO` | Exposición en años (fracción del año en vigor) |
| `ZONA` | Zona geográfica (A–E) |
| `POTENCIA` | Potencia del vehículo |
| `ANTIGUEDAD_VEHICULO` | Antigüedad del vehículo en años |
| `EDAD_CONDUCTOR` | Edad del conductor |
| `BONO` | Bonus-malus del asegurado |
| `MARCA` | Marca del vehículo (código) |
| `COMBUSTIBLE` | Tipo de combustible (D = Diesel, E = Eléctrico, etc.) |
| `DENSIDAD` | Densidad de población de la zona |
| `REGION` | Región administrativa |

### `siniestros.csv` — Siniestros reportados

| Campo | Descripción |
|-------|-------------|
| `ID` | Identificador único del siniestro |
| `NUMPOL` | Número de póliza (llave foránea) |
| `COD_COBERTURA` | Código de cobertura (`1RC`, `2DO`, etc.) |
| `MONTO` | Monto del siniestro |

### `BaseTrabajo.csv` — Base unificada

Join de `vigor` y `siniestros` filtrado a cobertura `1RC`. Contiene todas las columnas de `vigor` más `COD_COBERTURA`, `MONTO` y `num_sin` (conteo de siniestros por póliza).

---

##  Requisitos

```bash
pip install numpy pandas matplotlib statsmodels scipy patsy pyreadr
```

| Librería | Uso |
|----------|-----|
| `numpy` | Simulación y cálculo numérico |
| `pandas` | Manipulación de datos |
| `matplotlib` | Visualización |
| `statsmodels` | GLMs (Poisson, Binomial Negativa, Gamma) |
| `scipy` | Ajuste de distribuciones (Weibull) |
| `patsy` | Fórmulas de modelos con variables de referencia |
| `pyreadr` | Lectura de archivos `.rds` de R |

---

##  Uso

1. Clonar el repositorio:
   ```bash
   git clone https://github.com/<usuario>/Pricing.git
   cd Pricing
   ```
2. Instalar dependencias:
   ```bash
   pip install numpy pandas matplotlib statsmodels scipy patsy pyreadr
   ```
3. Ejecutar los notebooks en orden:
   - `Parte 1.ipynb` → genera la cartera simulada
   - `Parte 2.ipynb` → construye la base de trabajo y analiza dispersión
   - `Parte 3.ipynb` → modela frecuencia, severidad y prima pura

> Los notebooks deben ejecutarse con la carpeta `data/` en la misma ruta relativa.

---

##  Principales resultados

- El **GLM Poisson** recupera correctamente la estructura del riesgo simulado y resulta suficiente ante la sobredispersión observada (varianza/media = 1.41).
- El **GLM Gamma recortado** (p95) mejora significativamente el ajuste de severidad (ΔAIC ≈ 1,161), indicando que los siniestros catastróficos requieren tratamiento separado.
- La **prima pura** refleja la heterogeneidad del portafolio: el giro médico concentra la mayor prima por su combinación de alta frecuencia y alta severidad.
- El **índice de Gini** de −0.21 señala oportunidades de mejora en la discriminación del modelo para futuras calibraciones.
