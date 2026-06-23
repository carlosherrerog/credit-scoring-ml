# Sistema de Machine Learning para Credit Scoring

Este proyecto constituye mi Trabajo de Fin de Grado del Grado en Estadística.

## Descripción

Desarrollo de un sistema de ***credit scoring*** para la concesión de **préstamos**
**bancarios de carácter personal**, basado en datos reales de clientes de Caja España. 
El proyecto replica en Python el estudio de referencia de Fernando Mallo; realizado
originalmente en R, y lo extiende con modelos modernos de Machine Learning,
evaluados bajo el marco regulatorio de Basilea II.

El objetivo del trabajo es construir y validar un modelo capaz de estimar la
probabilidad de impago de cada solicitante a partir de su perfil financiero y,
con esa probabilidad, fundamentar la decisión de conceder o denegar el préstamo.
Se busca además que el modelo sea preciso (alta capacidad de discriminación
entre buenos y malos pagadores) e interpretable, de modo que cada decisión
pueda justificarse ante el cliente y ante el regulador, equilibrando el riesgo
asumido por la entidad con el volumen de operaciones aprobadas.

> **Nota:** los datos originales son confidenciales y no se incluyen en este
> repositorio.

## Punto de partida: el estudio de Fernando Mallo

El conjunto de datos original contenía 64 variables. Fernando Mallo, profesor e
investigador con estrecha vinculación a Caja España, depuró esa información y
preseleccionó las 25 variables predictoras de las que parte este trabajo. Sobre
esa misma muestra, Mallo desarrolló en R un modelo de credit scoring cuyo
modelo de referencia, una Regresión Logística con 12 predictores, alcanzó sobre
la muestra de validación un AUC de 0,9758, un estadístico de Kolmogórov-Smirnov
(KS) de 0,8897 y una tasa de precisión del 98,17 %. Estos valores constituyen la
referencia que este análisis busca igualar y, si es posible, superar.

## Cómo se ha procedido: aportación de este trabajo

La aportación de este trabajo es triple:

1. **Replicar en Python** el análisis que Mallo elaboró en R, reproduciendo su
   metodología y su modelo de referencia.

2. **Extenderlo con modelos de Machine Learning modernos** que él no aplicó,
   concretamente Random Forest y XGBoost, e incorporar interpretabilidad
   mediante valores SHAP.

3. **Demostrar que la metodología es generalizable**, reaplicando todo el
   procedimiento a un conjunto de datos público de credit scoring de clientes de
   Taiwán. No se reutiliza el modelo de Caja España, sino que se construye uno
   nuevo desde cero con esos datos, adaptando los pasos cuando es necesario (por
   ejemplo, ponderación de clases en lugar de sobremuestreo). Esto confirma que
   el flujo de trabajo es trasladable a otros contextos.

## Datos

El estudio se realiza sobre un conjunto de datos real e interno de la entidad
bancaria Caja España, facilitado en un contexto académico. Cada registro
corresponde a un cliente con préstamos vivos y recoge su comportamiento
financiero: saldos medios y mínimos de sus cuentas, ingresos recurrentes,
nivel y evolución de la deuda, antigüedad como cliente y de sus productos,
ratios de endeudamiento e incidencias de pago, entre otras.

- 73.210 clientes y 25 variables financieras preseleccionadas.
- Variable objetivo binaria: impago (1) frente a pago (0).
- Fuerte desbalanceo de clases: aproximadamente un 7,5 % de impagos.

Durante la modelización se detectaron dos de esas variables, el porcentaje
de contratos con incidencia y los meses en descubierto del último año, cuya
capacidad predictiva resultó ser una fuga de información: describen un
comportamiento de impago casi simultáneo al que mide la propia variable
objetivo, no un factor de riesgo previo a este. Ambas se excluyeron del
análisis final, que se desarrolla sobre las 28 variables restantes.

## Metodología

1. **Análisis exploratorio** de las variables y de su relación con el impago.
2. **Tratamiento de valores ausentes** mediante variables indicadoras de
   ausencia (*flags*) e imputación.
3. **Tratamiento del desbalanceo** con sobremuestreo estratificado (proporción
   1:2) y comparación con la ponderación de clases.
4. **Partición** estratificada en entrenamiento, validación y test (50/25/25).
5. **Modelización** y comparación de tres familias de modelos.
6. **Evaluación** sobre un conjunto de test reservado y **validación externa**
   sobre un conjunto de datos independiente.
7. **Interpretabilidad** del modelo final mediante valores SHAP.

## Modelos

- Regresión Logística (modelo de referencia).
- Regresión Logística con transformación *Weight of Evidence* (WoE).
- Random Forest.
- XGBoost.

## Resultados principales

| Modelo                     | AUC (test) | KS (test) | Error Tipo II |
|----------------------------|:----------:|:---------:|:-------------:|
| Regresión Logística        |   0,8886   |   0,6325  |     61,6 %    |
| Regresión Logística + WoE  |   0,9133   |   0,6909  |     53,9 %    |
| Random Forest              |   0,9397   |   0,7455  |     34,1 %    |
| **XGBoost**                | **0,9422** | **0,7630**|   **33,0 %**  |

El modelo de referencia de Mallo (AUC = 0,9758 en su muestra de validación)
incluye las dos variables señaladas más arriba como fuga de información. Al
excluirlas, criterio seguido en este trabajo, el AUC de XGBoost sobre el
conjunto de test cae de cerca de 0,99 a 0,9422: una cifra más alineada con lo
habitual en credit scoring (0,70-0,85) y con el AUC de 0,78 obtenido en la
validación externa sobre el conjunto de clientes de Taiwán.

## Interpretabilidad

El análisis con valores SHAP identifica las variables que más influyen en la
decisión del modelo, construye una curva de parsimonia (rendimiento frente a
número de variables) y permite explicar la concesión o denegación del préstamo
cliente a cliente, en línea con los requisitos de transparencia de Basilea II.

## Ámbito de aplicación y limitaciones

Este modelo se ha construido en exclusiva para los clientes de la entidad
bancaria de la que proviene el conjunto de datos (Caja España) y para su año de
referencia (2010). Cada modelo de credit scoring se ajusta a unas condiciones
concretas: una población de clientes, un entorno económico y un momento
temporal determinados. Por ello, aplicar este modelo en la actualidad o sobre
clientes de otra entidad sería metodológicamente erróneo, ya que esas
condiciones han cambiado. Un uso real exigiría reconstruir y recalibrar el
modelo con datos actuales de la población objetivo.

## Estructura

- `notebooks/`: análisis completo, en orden:
  - `01` Análisis exploratorio de datos (EDA).
  - `02` Preprocesamiento y particiones.
  - `03` Modelización (Regresión Logística, Random Forest, XGBoost).
  - `03b` Regresión Logística optimizada con transformación WoE.
  - `04` Evaluación sobre el conjunto de test (AUC, KS, Gini, matrices de confusión).
  - `04b` Interpretabilidad y selección de variables con SHAP.
  - `05` Validación externa.
- `reports/figures/`: gráficos de resultados.

## Tecnologías

Python 3, pandas, scikit-learn, imbalanced-learn, XGBoost, SHAP, matplotlib y seaborn.

## Instalación

```bash
python3 -m venv venv
source venv/bin/activate
pip install -r requirements.txt
```

## Autor

Carlos Herrero García. Grado en Estadística, Universidad de Salamanca.
