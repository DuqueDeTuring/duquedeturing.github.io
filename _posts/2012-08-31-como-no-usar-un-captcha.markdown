---
layout: post
title:  "Como NO hacer un Captcha"
date:   2012-08-31
categories: programming
---

El objetivo principal para el uso de un [CAPTCHA](https://es.wikipedia.org/wiki/Captcha) es la eliminación del abuso de una página Web por medio de programas, frecuentemente llamados [bots](https://es.wikipedia.org/wiki/Bots).
Después de utilizar en varias ocasiones la consulta civil en el sitio del [Tribunal Supremo de Elecciones de Costa Rica](https://www.tse.go.cr) (TSE), pensé estar sufriendo de una anomalía de mi memoria: el código CAPTCHA que me solicitaba la página me parecía familiar.

> Narrador: *No fue un déjà vu*.

El sitio tiene un conjunto predefinido de imágenes para el CAPTCHA, es decir, no son generados de forma dinámica y ni siquiera son un gran número.

> Narrador: Pero eso no es lo peor.

Veamos un ejemplo que me solicita la página.

El código que se solicita digitar es **VNC6G2** y este es el enlace a la imagen que muestran:
```html
<img id="picCaptcha" src="imagenes/VNC6G2.bmp" style="border-style:Dotted;height:56px;width:160px;border-width:0px;" /</code>&gt;
```

El nombre del archivo de la imagen para el CAPTCHA **tiene los caracteres del código que el usuario debe digitar**.

Es decir, es **trivial** hacer un programa que haga consultas en la página del TSE con sólo leer el nombre de la imagen que enlazan en el elemento con id *“picCaptcha”* y luego enviarlo en el campo *“txtcodigo”* del formulario junto con los datos de la persona que se quiere consultar.
