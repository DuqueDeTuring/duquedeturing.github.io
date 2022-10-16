---
layout: post
title: "Alerta de cambio en página Web con una línea en bash"
date:   2022-10-16
---

---

Desde hace mucho tiempo he querido escribir una herramienta personal para avisarme de cambios en el contenido de una página Web (claro que existen servicios que lo hacen pero quería programarlo).

Bueno, en un momento de lucidez, hace unas semanas se me ocurrió que no necesitaba escribir una herramienta: ¡tengo todo lo necesario en el shell y para hacerlo en una línea!...¡Y la base de datos consiste en un simple archivo de texto plano!

La idea es sencilla: guardar el hash de una página (o sección) y compararlo frecuentemente con el actual. 

### Caso ejemplo
Puede servir para monitorear -entre otros- sistemas y uptime pero para implementar un caso ejemplo vamos a monitorear periódicamente cualquier cambio en disponibilidad en páginas de un par de criaturas mitológicas de nuestro tiempo: el [Rasperry Pi Zero 2 W](https://www.canakit.com/raspberry-pi-zero-2-w.html|) y el [Arduino 33 BLE Sense](https://store-usa.arduino.cc/collections/sensors-environment/products/arduino-nano-33-ble-sense-with-headers).




### Solución
Se implementa en una línea de bash utilizando `curl`, `awk`, `sha256sum` y lo hacemos por etapas: generamos un archivo con cada url y su corresspondiente hash, revisamos cada línea de la lista con awk para generar una línea de bash que hace la comparación y que a su vez al ejecutarse generará otra línea de bash, que para el caso de que el hash cambie, esta última línea será la que origina la alerta.


**Primero**, inicializamos la lista ("base de datos") de los URLs a monitorear junto con su hash _sha 256_ en un archivo llamado wiff.db:


{% highlight bash %}
URL="https://www.canakit.com/raspberry-pi-zero-2-w.html"; echo "$URL|$(curl -L -s -A 'Chrome/5
1.0.2704.106' $URL | sha256sum) " > $HOME/wiff.db
URL="https://store-usa.arduino.cc/collections/sensors-environment/products/arduino-nano-33-ble-sense"; echo "$URL|$(curl -L -s -A 'Chrome/51.0.2704.106' $URL | sha256sum) " >> $HOME/wiff.db
{% endhighlight %}

El archivo va a tener algo como:

{% highlight bash %}
https://www.canakit.com/raspberry-pi-zero-2-w.html|dfe5283b78f17cd2fe6b97b6263804c3230cf07c95d3a07e1850fda57d787b1c  -
https://store-usa.arduino.cc/collections/sensors-environment/products/arduino-nano-33-ble-sense|32322fd6b11be0dd03f2a4fdeaf3147229b8c1eb7e502c246422a28576244f03  -
{% endhighlight %}

**Segundo**, definimos un *cron job* para que cada hora se revisen todos los URLs registrados en la base de datos:

{% highlight bash %}
0 */1 * * * cat $HOME/wiff.db  | awk -F "|" '{ print "if [ \"$(curl -L -s -A \x27Chrome/51.0.2704.106\x27 \x27" $1 "\x27 |sha256sum)\" == \x27"$2"\x27 ]; then echo \"\";else ALERTA; fi;" }'  | bash | bash
{% endhighlight %}

**Tercero**...¡no hay tercer paso!


## ¿Cómo funciona?
La base de datos es un simple archivo de texto en donde cada línea tiene el URL y su hash calculado, separados con un `|`.
Luego, la revisión se hace ejecutando el mismo cálculo para cada línea registrada haciendo uso de _awk_ que nos separa los campos y nos permite ejecutar algo para cada línea. Lo generado con el `print` en _awk_ lo ejecutamos con el primer `| bash` que realiza la comparación del nuevo hash contra el original, esta comparación (en el _if_) genera una nueva línea de bash que si el hash es el mismo (*then*) es una línea vacía pero si el hash es distinto nos genera una línea con la notificación (`$1` tiene el URL y `$2` el hash original). 

En mi caso esta notificación es un simple mensaje a un [MQTT](https://mqtt.org) broker ([mosquitto](https://mosquitto.org/)) pero puede ser cualquier otra cosa, por ejemplo: enviar un correo, un mensaje a una cola en SQS, invocar una función Lambda en AWS, encender una luz, mensaje a Pub/Sub, un shell script, etc.

La ejecución del curl y el cálculo del hash (que se ejecuta en un *subshell*) lo "envolvemos" en un *if* para compararlo con el valor guardado inicialmente que awk nos expone en el *$2* y si son diferentes se ejecuta el *else* con la parte que indiqué con "ALERTA". Esta parte es la ejecutada con el segundo `| bash` de la línea.

Viéndolo en capas, lo generado por _awk_ para el primer URL, en orden :

`if [ "$(curl -L -s -A 'Chrome/51.0.2704.106' 'https://www.canakit.com/raspberry-pi-zero-2-w.html'|sha256sum)" == 'dfe5283b78f17cd2fe6b97b6263804c3230cf07c95d3a07e1850fda57d787b1c  -' ]; then echo "";else ALERTA; fi;`

Esto es lo primero interpretado por el primer `| bash`, se calcula el hash del URL y se compara con el inicial que se guardó en el archivo de texto, si el hash es el mismo la condición retorna `echo ""` que será el _input_ para el segundo y último `| bash`. 

Si el resultado de la condición anterior es que el hash es distinto, el _else_ retorna `ALERTA`, esto será interpretado por el segundo y último `| bash`.


## A considerar:
- El agent en _curl_ (parámetro -A) es importante porque algunos servidores ignoran el agent por defecto de _curl_ (algo como: *user-agent: curl/7.81.0*) y retornan un documento vacío[^1].
- En mi caso la alerta fue trivial de enviar ya que el [MQTT broker](https://mosquitto.org/) que uso para [Home Assistant](https://www.home-assistant.io/) sirve perfectamente. Me permite un simple `mosquitto_pub` para enviar un mensaje a un tópico y, configurando una pequeña [automatización](https://gist.github.com/DuqueDeTuring/55881791e65a31fda148c8ac85aabfb7) en Home Assistant recibir una notificación en el celular. 
- Muchas páginas de tiendas generan ids dinámicos dentro del html cada vez que se consultan y el caso ejemplo no es la excepción, la solución es tan simple como agregar un `| grep LOQUEVAACAMBIAR` antes del `| sha256sum` para extraer alguna sección que sabemos cambiará sólo cuando lo que nos interesa sea modificado por el sitio. 
- Uno de mis *cron jobs* para revisión frecuente de una de las listas quedó así:

{% highlight bash %}
0 */1 * * * cat $HOME/wiff.db  | awk -F "|" '{ print "if [ \"$(curl -L -s -A \x27Chrome/51.0.2704.106\x27 \x27" $1 "\x27|grep -i \x27sold out\x27|sha256sum)\" == \x27"$2"\x27 ]; then echo \"\";else echo mosquitto_pub -h MQTT_IP -u USUARIO -P PASSWD -t url_updated -m \""$1"\"; fi;" }'  | bash | bash
{% endhighlight %}


---

[^1]: wow, qué protección más avanzada...
