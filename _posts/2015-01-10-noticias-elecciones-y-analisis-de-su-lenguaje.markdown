---
layout: post
title:  "Noticias, elecciones y análisis de su lenguaje"
date:   2015-01-10
---

Febrero 2014 fue mes de las elecciones presidenciales en Costa Rica. Poco más de un año antes y luego de terminar de leer un excelente libro que, entre otras cosas, analiza la influencia de los medios de comunicación para moldear la opinión pública (como herramientas de los grupos de poder), se me ocurrió que sería interesante poder realizar un análisis de las noticias publicadas durante el año electoral.

Así que armado con un [simple programa](https://github.com/duquedeturing/captura-feeds), desde el 26 de Julio de 2013* en adelante realicé la captura diaria de todas las noticias publicadas en el sitio Web del periódico [La Nación](http://www.nacion.com), guardando los datos en simples archivos para su análisis posterior.

Este es el primer artículo de varios sobre la captura y la aplicación de herramientas de análisis de lenguaje natural al texto de las noticias.
Fuente

La extracción del texto se hizo del RSS del sitio Web: [http://www.nacion.com/rss.html](http://www.nacion.com/rss.html). La información del feed la guardé en archivos .json (2 por día) con diferentes llaves para almacenar metadata que publican en el RSS.

Cada noticia existe bajo una categoría:

- Nacionales
- Internacionales
- Economía
- Sucesos
- Entretenimiento
- Portada
- Opinión

Y cada noticia lleva:

- Título
- Fecha de publicación
- Enlace
- Descripción
- Noticia

Teniendo todas las noticias clasificadas por categoría, permite hace preguntas como:
– ¿Cuáles fueron los temas considerados suficientemente importantes como para estar en las portadas del periódico?
– En la sección de economía, ¿qué fue lo que más publicó el periódico?

El pequeño programa se ejecuta diariamente de forma automática con un simple cron job:

```bash
java -jar captura-feeds-0.1.1-SNAPSHOT-standalone.jar prensky/noticias_$(date -u "+%Y-%m-%d_%H_%M_%S").json
```

Un ejemplo de los archivos generados se puede ver en: [https://gist.github.com/duquedeturing/a856e722439e40684b10](https://gist.github.com/duquedeturing/a856e722439e40684b10)