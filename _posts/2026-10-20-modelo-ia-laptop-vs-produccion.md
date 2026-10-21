---
layout: post
title: "Por qué un modelo de IA que funciona en tu laptop falla en producción"
date: 2026-10-20
categories: [Inteligencia Artificial]
excerpt: "Un modelo con buena precisión en el cuaderno no es un sistema en producción. Estas son las cuatro brechas que más proyectos de IA arruinan."
---

Un modelo con 92 % de precisión en tu laptop no está listo para nadie más. Entre el cuaderno de pruebas y un sistema que usan personas reales hay una distancia que casi nunca se planifica, y ahí es donde se pierden la mayoría de los proyectos de IA.

## 1. Los datos cambian

En desarrollo entrenas con un conjunto fijo y limpio. En producción llegan datos nuevos, incompletos o con formatos distintos, y con el tiempo la realidad se aleja de lo que el modelo aprendió. Esto se llama *deriva de datos* y degrada el rendimiento sin generar ningún error visible.

**Qué hacer:** guarda las entradas que recibe el modelo y compara periódicamente su distribución con la del entrenamiento.

## 2. Nadie lo está monitoreando

Un servidor caído se nota. Un modelo que empieza a equivocarse, no. Sigue respondiendo con total seguridad, pero cada vez peor.

**Qué hacer:** monitorea tres cosas desde el primer día:

- Salud técnica: latencia, errores y consumo de recursos.
- Calidad de las predicciones, con una muestra revisada por una persona.
- Cambios en los datos de entrada.

## 3. El entorno no es el mismo

"En mi máquina funciona" también aplica a la IA: versiones distintas de librerías, otra versión de Python, otra GPU o ninguna. Pequeñas diferencias numéricas pueden cambiar resultados.

**Qué hacer:** empaqueta el modelo con sus dependencias fijas en un contenedor y prueba en un entorno idéntico al de producción antes de publicar.

## 4. No hay plan de reentrenamiento

Muchos equipos entregan el modelo como si fuera un producto terminado. Es más parecido a un cultivo: requiere mantenimiento. Si nadie sabe cuándo ni cómo se reentrena, el modelo envejece hasta que alguien se queja.

**Qué hacer:** define desde el inicio qué señal dispara un reentrenamiento, quién lo ejecuta y cómo se valida la versión nueva antes de reemplazar la anterior.

## Una regla práctica

Antes de celebrar la precisión del modelo, responde estas preguntas:

1. ¿Qué pasa si los datos de mañana son distintos a los de hoy?
2. ¿Cómo me enteraría de que empezó a fallar?
3. ¿Puedo reproducir exactamente este entorno en otro servidor?
4. ¿Quién es responsable del modelo dentro de seis meses?

Si alguna respuesta es "no sé", el proyecto todavía no está listo para producción, por bueno que sea el modelo.

## Conclusión

Entrenar el modelo es la parte visible del trabajo. Lo que decide si un proyecto de IA sobrevive es la infraestructura, el monitoreo y el mantenimiento que lo rodean, que es justo donde infraestructura y ciencia de datos se encuentran.
