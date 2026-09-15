# Medición de Incrementalidad (Lift) mediante Series Temporales y Modelos Contrafactuales

Este repositorio contiene un marco analítico *end-to-end* desarrollado en Python para evaluar el efecto incremental (*Lift*) de campañas de marketing e intervenciones de negocio sobre series temporales mediante la estimación de escenarios contrafactuales.

## 📌 Contexto y Problema de Negocio
Evaluar el verdadero impacto causal de una campaña publicitaria u operativa requiere comparar lo observado contra lo que **habría sucedido en ausencia de la intervención** (escenario contrafactual). 

Las métricas tradicionales suelen incurrir en sesgos de atribución. Este proyecto implementa y compara cuatro metodologías avanzadas de inferencia causal para garantizar estimaciones precisas y estadísticamente sólidas.

---

## 🛠️ Metodologías Implementadas

1. **Synthetic Control (Ajustado por Escala)**: Optimización convexa de pesos sobre el *donor pool* ($w_i \ge 0, \sum w_i = 1$) con corrección de sesgo pre-campaña para corregir brechas de nivel base.
2. **Structural Time Series (BSTS-lite / OLS)**: Regresión estructural de espacio de estados que incorpora un intercepto no restringido para absorber diferencias de nivel constante.
3. **Prophet (Additive Model)**: Descomposición aditiva de tendencia y estacionalidades (semanal/anual) entrenada exclusivamente en la fase pre-intervención.
4. **Inferencia Bayesiana (PyMC)**: Muestreo MCMC sobre las distribuciones posterior de los pesos e intercepto, cuantificando la incertidumbre mediante Intervalos de Credibilidad al 95%.

---

## 📊 Resultados y Salud de los Modelos

| Método | Efecto Diario Estimado | Pre-MAPE (%) | Pre-Bias | Criterio de Selección |
| :--- | :---: | :---: | :---: | :--- |
| **Ground Truth (Real)** | **450.00** | **0.00%** | **0.00** | Base de Comparación |
| **Synthetic Control (Ajustado)** | 460.52 | 0.67% | 0.00 | Válido (Alta Interpretabilidad) |
| **BSTS-lite (OLS)** | 457.71 | 0.67% | 0.00 | Válido (Robusto) |
| **Prophet** | 482.10 | 2.15% | +12.30 | Válido (Sensible a tendencia) |
| **PyMC (Bayesiano)** | **458.20** | **0.67%** | **0.00** | **Recomendado (Manejo de Incertidumbre)** |

> **Mecanismo Dinámico**: El notebook incluye un algoritmo (`select_best_model_dynamically`) que descarta automáticamente modelos con `Pre-MAPE > 3.0%` y prioriza enfoques Bayesianos para la toma de decisiones basada en riesgo.

---

## 🚀 Cómo Ejecutar este Proyecto

### 1. En Google Colab (Recomendado)
Puedes ejecutar directamente el cuaderno interactivo en Google Colab:
[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanmanser/causal-impact-time-series/blob/main/notebooks/incrementalidad_series_temp.ipynb)

### 2. Entorno Local
```bash
# Clonar repositorio
git clone [https://github.com/juanmanser/causal-impact-time-series.git](https://github.com/juanmanser/causal-impact-time-series.git)
cd causal-impact-time-series

# Crear y activar entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install scipy scikit-learn statsmodels matplotlib pymc arviz prophet plotly
