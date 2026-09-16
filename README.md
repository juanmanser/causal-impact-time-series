# Medición de Incrementalidad (Lift) mediante Series Temporales y Modelos Contrafactuales

Este repositorio contiene un marco analítico *end-to-end* desarrollado en Python para evaluar el efecto incremental (*Lift*) de campañas de marketing e intervenciones de negocio sobre series temporales mediante la estimación de escenarios contrafactuales.

## 📌 Contexto y Problema de Negocio

Evaluar el verdadero impacto causal de una campaña publicitaria u operativa requiere comparar lo observado contra lo que **habría sucedido en ausencia de la intervención** (escenario contrafactual).

Las métricas tradicionales suelen incurrir en sesgos de atribución. Este proyecto implementa y compara cuatro metodologías avanzadas de inferencia causal para garantizar estimaciones precisas y estadísticamente sólidas.

---

## 🛠️ Metodologías Implementadas

1. **Synthetic Control (Ajustado por Intercepto)**: Optimización convexa de pesos sobre el *donor pool* ($w_i \ge 0, \sum w_i = 1$) con un intercepto estimado dentro de la optimización pre-campaña para absorber diferencias de nivel base. Incluye *placebo test* por permutación para asignar un p-valor al efecto estimado.
2. **Structural Time Series (BSTS-lite / OLS)**: Regresión estructural con intercepto no restringido y coeficientes libres; absorbe diferencias de nivel constante e interacciones entre donantes.
3. **Prophet (Additive Model)**: Descomposición aditiva de tendencia y estacionalidades, entrenada exclusivamente en la fase pre-intervención. La estacionalidad anual se habilita dinámicamente cuando hay al menos 730 días de historia.
4. **Inferencia Bayesiana (PyMC)**: Muestreo MCMC con `target_accept` elevado y chequeo de diagnósticos (divergencias, R-hat, ESS). Se reporta el Intervalo de Credibilidad al 95% y se marca explícitamente como no fiable si la convergencia falla.

---

## 📊 Resultados y Salud de los Modelos

Corrida de referencia sobre panel sintético: 365 días pre-campaña, 28 días post-campaña, 6 donantes, efecto inyectado real de **450.00 diarios / 12,600.00 totales**.

| Método | Efecto Diario | Efecto Total | Lift Acumulado (%) | Error vs GT (%) | Pre-MAPE (%) | Incertidumbre |
| :--- | :---: | :---: | :---: | :---: | :---: | :--- |
| **Ground Truth (Real)** | **450.00** | **12,600.00** | — | — | — | Base de Comparación |
| **Synthetic Control (Intercepto)** | 510.22 | 14,286.24 | +6.07% | +13.38% | 0.91% | Placebo p = 0.500 |
| **BSTS-lite (OLS)** | 460.55 | 12,895.45 | +5.45% | +2.34% | 0.67% | N/A |
| **Prophet** | 389.48 | 10,905.35 | +4.57% | -13.45% | 0.59% | [8,726 – 13,135] |
| **PyMC (Bayesiano)** | 458.42 | 12,835.65 | +5.42% | +1.87% | 0.69% | [12,546 – 13,129] |

**Lectura rápida:**

- **PyMC** y **BSTS-lite** son los que mejor recuperan el efecto real (error < 2.5%).
- **Prophet** subestima el efecto en ~13%, consistente con la ausencia de estacionalidad anual con solo 365 días de historia.
- **Synthetic Control** sobreestima ~13%, y el placebo p = 0.500 indica que el efecto no se distingue estadísticamente del ruido estructural del pool de donantes.
- El **lift acumulado** ronda entre +4.57% y +6.07% según el método, lo que da una banda de decisión útil para stakeholders.

> **Robustez**: el rango inter-métodos del efecto total es 26.56% de la media, lo que indica **consenso bajo**. Recomendación operativa: para decisiones de alto impacto, priorizar los métodos con incertidumbre formal (PyMC con convergencia, o Prophet con su intervalo nativo).

---

## 🎯 Selección Dinámica del Modelo

El notebook incluye una función `select_best_model_dynamically` que **no hardcodea ninguna preferencia por método**. En su lugar, construye un score compuesto:

score = Pre-MAPE + 10 · |Pre-Bias| + 0.5 · Desviación vs Consenso (%)


- **Filtros duros**: descarta modelos con `Pre-MAPE > 3.0%` o `|Pre-Bias| > 1.0`.
- **Penalización por consenso**: castiga alejarse de la mediana del efecto total entre métodos.
- **Penalización por convergencia**: si PyMC no converge, se suma +5 al score.
- **Fallback seguro**: si ningún modelo pasa los filtros, se elige el de menor score sin filtro.

En esta corrida, el modelo seleccionado fue **BSTS-lite (OLS)** con score = 0.782, impulsado por su MAPE bajo (0.67%) y bias nulo. Si el objetivo del análisis fuera reportar incertidumbre formal, PyMC queda segundo con el mismo MAPE efectivo y un IC bayesiano ya calculado.

---

## 🚀 Cómo Ejecutar este Proyecto

### 1. En Google Colab (Recomendado)

Puedes ejecutar directamente el cuaderno interactivo en Google Colab:

[![Open In Colab](https://colab.research.google.com/assets/colab-badge.svg)](https://colab.research.google.com/github/juanmanser/causal-impact-time-series/blob/main/lift_time_series_multi_meth_py.ipynb)

### 2. Entorno Local

```bash
# Clonar repositorio
git clone https://github.com/juanmanser/causal-impact-time-series.git
cd causal-impact-time-series

# Crear y activar entorno virtual
python -m venv venv
source venv/bin/activate  # En Windows: venv\Scripts\activate

# Instalar dependencias
pip install -r requirements.txt
