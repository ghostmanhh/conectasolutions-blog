---
layout: post
title: "El problema que nadie te cuenta sobre poner un modelo de ML en producción"
subtitle: "De Jupyter Notebook a sistema real: la brecha que destruye proyectos de IA"
date: 2025-04-18
categories: [Inteligencia Artificial]
tags: [machine learning, MLOps, producción, Python, FastAPI]
author: "Conecta Solutions"
read_time: 9
excerpt: "Un modelo que funciona en tu laptop no es un modelo en producción. La diferencia entre ambos destruye más proyectos de IA que la falta de datos o un algoritmo equivocado."
---

Hemos visto este escenario demasiadas veces: el equipo de datos entrena un modelo que predice con 92% de precisión en el conjunto de prueba. Todos están emocionados. Tres meses después, el modelo nunca llegó a producción.

No por falta de talento. Por falta de ingeniería.

## La brecha de producción

Un Jupyter Notebook es un artefacto de exploración. No es software. La transición de "modelo que funciona" a "sistema que sirve predicciones a usuarios reales" requiere resolver problemas que los cursos de ML raramente mencionan:

- ¿Cómo sirves predicciones con latencia aceptable?
- ¿Qué pasa cuando los datos de entrada tienen formato incorrecto?
- ¿Cómo sabes si el modelo empezó a degradarse?
- ¿Cómo actualizas el modelo sin interrumpir el servicio?

## Una arquitectura mínima viable

Esta es la arquitectura que usamos para proyectos pequeños y medianos:

```
[Fuente de datos] → [Pipeline de features] → [Modelo] → [API REST] → [Cliente]
                                                  ↕
                                          [Monitoreo de drift]
```

### 1. Serializa el modelo correctamente

```python
import joblib
from sklearn.pipeline import Pipeline

# Incluye el preprocesamiento EN el pipeline
# Nunca serialices solo el modelo
full_pipeline = Pipeline([
    ('scaler', StandardScaler()),
    ('model', RandomForestClassifier())
])

full_pipeline.fit(X_train, y_train)
joblib.dump(full_pipeline, 'model_v1.pkl')
```

Empacar el preprocesamiento con el modelo elimina una categoría entera de bugs en producción.

### 2. Sirve el modelo con FastAPI

```python
from fastapi import FastAPI
from pydantic import BaseModel
import joblib
import numpy as np

app = FastAPI()
model = joblib.load("model_v1.pkl")

class PredictionRequest(BaseModel):
    features: list[float]

@app.post("/predict")
def predict(request: PredictionRequest):
    X = np.array(request.features).reshape(1, -1)
    prediction = model.predict(X)[0]
    probability = model.predict_proba(X)[0].max()
    return {
        "prediction": int(prediction),
        "confidence": round(float(probability), 4),
        "model_version": "v1"
    }
```

### 3. Monitorea el drift desde el primer día

El drift es cuando la distribución de los datos reales empieza a alejarse de los datos con los que entrenaste. El modelo sigue respondiendo — pero sus predicciones empiezan a ser incorrectas.

Herramientas útiles: **Evidently AI** para reportes de drift, **MLflow** para tracking de experimentos y versiones de modelos.

## Lo que más falla en proyectos reales

**La calidad de los datos de entrada.** En el notebook usas un CSV limpio. En producción, el dato viene de un formulario web llenado por un humano apresurado. Debes validar y sanitizar cada entrada.

**La falta de versionado del modelo.** Si el modelo en producción falla, ¿puedes hacer rollback al anterior en 5 minutos? Si la respuesta es no, tienes un problema de ingeniería, no de data science.

**No medir nada.** ¿Cuántas predicciones por día hace tu modelo? ¿Cuál es la distribución de las clases predichas? Sin métricas, no sabes cuándo algo salió mal hasta que ya es tarde.

El objetivo no es tener el modelo más sofisticado. Es tener un sistema confiable que entrega valor todos los días.
