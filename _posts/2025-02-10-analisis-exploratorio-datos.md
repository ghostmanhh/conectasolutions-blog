---
layout: post
title: "Análisis exploratorio de datos: el paso que todos quieren saltarse"
subtitle: "Por qué el EDA bien hecho te ahorra semanas de trabajo más adelante"
date: 2025-02-10
categories: [Ciencia de Datos]
tags: [EDA, Python, pandas, análisis, datos]
author: "Conecta Solutions"
read_time: 8
excerpt: "El análisis exploratorio de datos (EDA) no es el paso aburrido antes del modelo. Es donde se encuentran los problemas que destruirían tu análisis si los ignoras."
---

El patrón que vemos repetirse: el equipo quiere el modelo predictivo para ayer. El EDA se convierte en "correr algunas estadísticas básicas y ver si algo explota". Tres semanas después, el modelo tiene un desempeño terrible y nadie sabe por qué.

Spoiler: el problema estaba en los datos, y el EDA lo habría encontrado en horas.

## Qué es realmente un buen EDA

El EDA no es una serie de pasos mecánicos. Es un proceso de generación de preguntas sobre tus datos. Cada hallazgo debe llevar a más preguntas.

### Estructura antes que estadísticas

```python
import pandas as pd
import numpy as np

df = pd.read_csv('datos_empresa.csv')

# Lo primero: entender qué tienes
print(df.shape)           # Filas × columnas
print(df.dtypes)          # Tipos de datos
print(df.isnull().sum())  # Valores faltantes por columna
print(df.duplicated().sum())  # Filas duplicadas

# Estadísticas descriptivas, pero mira los percentiles
print(df.describe(percentiles=[.01, .05, .25, .5, .75, .95, .99]))
```

Los percentiles 1 y 99 revelan outliers que el promedio esconde.

### Valores faltantes — no asumas, investiga

```python
# ¿Los datos faltantes son aleatorios o tienen patrón?
missing = df[df['columna_importante'].isnull()]
print(missing['otra_columna'].value_counts())

# Visualización rápida del patrón de missing
import missingno as msno
msno.matrix(df)
```

Si los datos faltantes no son aleatorios (MNAR — Missing Not At Random), imputar con la media es incorrecto. Necesitas entender POR QUÉ faltan.

### Distribuciones — histogramas antes que modelos

```python
import matplotlib.pyplot as plt

fig, axes = plt.subplots(2, 3, figsize=(15, 8))
columnas_numericas = df.select_dtypes(include=np.number).columns[:6]

for ax, col in zip(axes.flat, columnas_numericas):
    df[col].hist(ax=ax, bins=50, color='#1A4F8C', alpha=0.7)
    ax.set_title(col)
    ax.set_xlabel('')

plt.tight_layout()
plt.savefig('distribuciones.png', dpi=150)
```

Una distribución bimodal puede indicar que tienes dos poblaciones mezcladas. Una distribución con cola muy larga puede requerir transformación logarítmica.

### Correlaciones — pero no las confundas con causalidad

```python
# Matriz de correlación
corr_matrix = df.select_dtypes(include=np.number).corr()

import seaborn as sns
plt.figure(figsize=(10, 8))
sns.heatmap(corr_matrix,
            annot=True, fmt='.2f',
            cmap='Blues',
            center=0,
            square=True)
plt.title('Matriz de correlación')
plt.tight_layout()
```

Correlación alta entre features de entrada es multicolinealidad — un problema para modelos lineales.

## Los hallazgos más comunes que salvan proyectos

En proyectos reales con clientes en Ecuador, estas son las cosas que el EDA encuentra antes de que arruinen el modelo:

1. **Fechas con formato inconsistente** — `01/05/2024` y `2024-05-01` en la misma columna
2. **Valores "0" que realmente son datos faltantes** — especialmente en campos de ventas o inventario
3. **Outliers legítimos vs. errores de captura** — una venta de $1.000.000 puede ser real o un error de digitación
4. **Leakage** — una columna que en teoría es un predictor pero en realidad contiene información del futuro
5. **Clases desbalanceadas** — cuando el 97% de registros es "normal" y el 3% es lo que te interesa predecir

## El entregable del EDA

Un EDA bien hecho produce un documento (puede ser un notebook limpio) que responde:
- ¿Qué datos tenemos y de qué calidad?
- ¿Qué transformaciones necesitan las variables?
- ¿Qué features tienen potencial predictivo?
- ¿Hay problemas que requieren más datos o limpieza antes de modelar?

Sin ese documento, el modelado es una apuesta.
