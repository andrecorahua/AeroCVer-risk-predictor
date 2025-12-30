# AeroCVer Risk Predictor ✈️🦉

Sistema de inteligencia artificial para la **predicción y monitoreo de riesgos operacionales** en la industria aeronáutica, basado inicialmente en incidentes de impactos con fauna (Bird Strikes).

El proyecto sirve como prototipo de la plataforma **AeroCVer**, pensada para adaptarse a distintos contextos operacionales (aviación comercial, helicópteros de carga, etc.).

---

## 🚀 Estado del Proyecto

**Prototipo Funcional (MVP)**

El sistema actualmente es capaz de:

1. **Procesar datos históricos** de incidentes de Bird Strikes (FAA/NTSB).
2. **Predecir la probabilidad de daño físico** en una aeronave tras un impacto.
3. **Detectar incidentes “anómalos”** que concentran un riesgo significativamente mayor.
4. Exponer una **función de inferencia** que, dado un vuelo de ejemplo, devuelve:
   - Probabilidad estimada de daño.
   - Clasificación final (riesgo con umbral ajustado).

---

## 📊 Resultados del Modelo de Clasificación

- **Algoritmo base:** `RandomForestClassifier`.
- **Dataset:** ~25,000 registros de incidentes reales (`bird_strikes_clean.csv`).
- **Variable objetivo:** `IsSevere` (1 = Caused damage, 0 = No damage).

### Desempeño con umbral estándar (0.50)
- Clase 0 (Sin daño): F1 ≈ 0.95
- Clase 1 (Con daño): F1 ≈ 0.28, Recall ≈ 0.19  
  → Modelo muy bueno para detectar “no daño”, pero conservador para daño.

### Desempeño con umbral ajustado (0.30)
Al reducir el umbral de decisión a **0.30**:

- Clase 1 (Con daño): Recall sube a ≈ **0.41** (más del doble),
- A costa de aumentar el número de falsas alarmas (lo cual es aceptable en contextos de seguridad).

Este ajuste convierte el modelo en una **herramienta más sensible a incidentes de alto riesgo**, priorizando la seguridad operativa frente a la comodidad operacional.

---

## 🔍 Resultados de Detección de Anomalías

Se utilizó `IsolationForest` para encontrar el ~2% de casos más inusuales:

- **Proporción de daño grave (`IsSevere`) en datos normales:** ~**8.9%**
- **Proporción de daño grave en anomalías detectadas:** ~**44.2%**

Es decir, los incidentes marcados como anómalos concentran un **riesgo de daño 5 veces mayor** que el resto.

Patrones observados en anomalías:
- Alta presencia en fases críticas como **Climb**, **Approach** y **Descent**.
- Algunos aeropuertos (ej. `BALTIMORE WASH INTL`) aparecen con frecuencia anómala.

Esto demuestra que el módulo de anomalías puede actuar como **sistema de alerta temprana** para priorizar la revisión de vuelos o incidentes inusuales.

---

## 🧱 Estructura del Proyecto

```text
aerosafe-risk-predictor/
│
├── data/
│   ├── raw/
│   │   └── Bird_strikes.csv               # Dataset original
│   └── processed/
│       ├── bird_strikes_clean.csv         # Dataset limpio y preprocesado
│       └── rf_model_risk.pkl              # Pipeline completo (preprocesador + RF)
│
├── notebooks/
│   ├── 01_exploracion_datos_publicos.ipynb
│   │   # EDA inicial: distribución de variables, análisis visual y estadísticas.
│   ├── 02_preprocesamiento_base.ipynb
│   │   # Limpieza, tratamiento de nulos, creación de IsSevere, Month, Year, etc.
│   ├── 03_modelos_anomalias_pyod.ipynb
│   │   # Detección de anomalías con IsolationForest y análisis de casos raros.
│   └── 04_modelos_clasificacion_basica.ipynb
│       # Entrenamiento del Random Forest, ajuste de umbral y demo de predicción.
│
├── src/
│   └── (espacio reservado para futuros módulos de API / integración)
│
├── README.md
└── requirements.txt

⚙️ Requisitos e Instalación

Requisitos básicos:

Python 3.x
pip configurado en tu entorno

Instala las dependencias principales con:

pip install scikit-learn pandas numpy matplotlib seaborn joblib


(Dependencias adicionales pueden estar listadas en requirements.txt.)

▶️ Cómo Reproducir los Resultados
Clonar el repositorio (o descargar el proyecto en tu máquina).
Asegurarte de que data/raw/Bird_strikes.csv existe.
Ejecutar los notebooks en este orden recomendado:
01_exploracion_datos_publicos.ipynb
02_preprocesamiento_base.ipynb → genera bird_strikes_clean.csv
03_modelos_anomalias_pyod.ipynb → análisis de anomalías
04_modelos_clasificacion_basica.ipynb → modelo de riesgo y demo de inferencia
El modelo final se guarda automáticamente como:
data/processed/rf_model_risk.pkl

Dentro de 04_modelos_clasificacion_basica.ipynb se incluye un ejemplo de uso:

Definición de un vuelo de ejemplo (tipo de aeronave, aeropuerto, fase de vuelo, etc.).
Cálculo de probabilidad de daño y clasificación con umbral 0.30.
🌍 Adaptación al Contexto de Helicópteros de Carga (Perú)

Aunque este prototipo se entrena con datos de Bird Strikes de la aviación en general, la arquitectura está pensada para ser agnóstica al tipo de aeronave:

El mismo pipeline de preprocesamiento + clasificación + anomalías puede aplicarse a:
Telemetría FOQA/FDR de helicópteros de carga.
Historial de incidentes y reportes internos de operadores peruanos.
Basta con:
Ajustar el conjunto de variables de entrada (altitud, velocidad, peso, configuración, etc.).
Volver a entrenar el pipeline con los datos del operador.
Recalibrar el umbral de riesgo según la tolerancia operacional de la compañía.

De este modo, AeroCVer Risk Predictor puede evolucionar hacia una herramienta de gestión de riesgo operacional en tiempo casi real para operaciones de helicópteros de carga en entornos complejos como la selva peruana.