---
layout: post
title:  "cURL, grep y awk al rescate"
date:   2013-03-04
categories: web unix
---
<h2>Problema</h2>
Teniendo una lista de números de guías de envíos de Correos de Costa Rica y sin tener acceso a un API de consultas: ¿cómo extraer la información de fechas de entrega desde el sitio Web del courier rápidamente?[^1]

<h2>Solución</h2>
Usando herramientas que son parte de una instalación típica de Linux o BSD, podemos fácilmente generar los resultados en un sólo archivo tipo CSV: *curl*, *grep* y *awk* serán suficientes.

## POST

Observando el comportamiento del <a href="https://correos.go.cr/rastreo/RastreoEnvios.html">formulario de rastreo</a> (por ejemplo con <a href="https://getfirebug.com/">Firebug</a>) del sitio de CorreosCR en <i>https://www.correos.go.cr/rastreo/RastreoEnvios.html</i>, la consulta genera un POST al URL <i>https://www.correos.go.cr/rastreo/tracking2.asp</i> con los siguientes parámetros:

    Envio_id
    Consultar

El parámetro que obviamente nos interesa es el que lleva el número de guía, es decir: Envio_id. Con esta información ya podemos generar la solicitud, con una guía de ejemplo EE123456 utilizaremos curl para generar la solicitud:

	curl -s  --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp

El POST anterior nos da un HTML que contiene los datos de entrega que nos interesa, la parte de de interés es la que contiene las fechas:

```
<TABLE WIDTH= 100%> 
<tr>
<td bgcolor="#559F00" width="170px" ><strong><font color="#fffafa" face="verdana,sans-serif" size="2">
<a href="ListaEnviosAnalisis_1.asp?order=Envio_ID"></a>&nbsp;Fecha - Hora</font></strong></td>

<td bgColor="#559F00"><strong><font color="#fffafa" face="verdana,sans-serif" size="2">
<a href="ListaEnviosAnalisis_1.asp?order=Fecha_ingreso"></a>&nbsp;Sucursal</font></strong></td>

<td bgColor="#559F00"><strong><font color="#fffafa" face="verdana,sans-serif" size="2">
<a href="ListaEnviosAnalisis_1.asp?order=Entrega_fecha"></a>&nbsp;Evento</font></strong></td>

<td bgColor="#559F00"><strong><font color="#fffafa" face="verdana,sans-serif" size="2">
<a href="ListaEnviosAnalisis_1.asp?order=Entrega_fecha"></a>&nbsp;Comentario</font></strong></td>
</tr>
<TR><TD><FONT face=verdana,sans-serif size=2>1/3/2013 3:49:05 PM</TD><TD>EMS COSTA RICA</TD><TD>Depósito/ Recogida</TD><TD>Registro del envío</TD></TR><TR><TD bgColor=#dcdcdc><FONT face=verdana,sans-serif size=2>1/4/2013 7:09:31 AM</TD><TD bgColor=#dcdcdc>OFICINA HEREDIA</TD><TD bgColor=#dcdcdc>Llegada a la oficina distribuidora</TD><TD bgColor=#dcdcdc></TD></TR><TR><TD><FONT face=verdana,sans-serif size=2>1/4/2013 2:08:00 PM</TD><TD>OFICINA HEREDIA</TD><TD>Entrega Final</TD><TD>Entregado a : OSCAR OSCAR</TD></TR><TR><TD>&nbsp;</TD></TR></TABLE>
```
{: .language-html}

Bien, tenemos entonces la página con los datos de fechas de recolección y entrega de una guía en particular, ¿y ahora qué?

## Aislamiento de la información con grep

Por suerte, los datos que necesitamos extraer se encuentran todos en una misma línea del HTML, tal y como podemos ver en el bloque anterior con los resultados, eso nos permite filtrar esa línea con un simple **grep**[^2]:

	curl -s  --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a"

Con la línea anterior obtenemos:

```
<TR><TD><FONT face=verdana,sans-serif size=2>1/3/2013 3:49:05 PM</TD><TD>EMS COSTA RICA</TD><TD>Depósito/ Recogida</TD><TD>Registro del envío</TD></TR><TR><TD bgColor=#dcdcdc><FONT face=verdana,sans-serif size=2>1/4/2013 7:09:31 AM</TD><TD bgColor=#dcdcdc>OFICINA HEREDIA</TD><TD bgColor=#dcdcdc>Llegada a la oficina distribuidora</TD><TD bgColor=#dcdcdc></TD></TR><TR><TD><FONT face=verdana,sans-serif size=2>1/4/2013 2:08:00 PM</TD><TD>OFICINA HEREDIA</TD><TD>Entrega Final</TD><TD>Entregado a : OSCAR OSCAR</TD></TR><TR><TD>&nbsp;</TD></TR></TABLE>

```
{: .language-html}

Cada una de las fechas está dentro de una columna del HTML, es decir, dentro de tags TD, lo cual es un problema perfecto para awk.


## Extracción de datos con awk

**awk**, un utilitario creado originalmente en los laboratorios Bell en los 70s nos da las facilidades de separar en registros patrones de búsqueda. Por medio del parámetro **-F** le indicamos lo que vamos a considerar como delimitadores para la aplicación de la expresión regular:

	curl -s  --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $0}' 

Con el **print $0** anterior imprimimos todo, pero lo que nos interesa son los registros encontrados por awk que contienen las fechas, si vemos el HTML separado por los tags TD encontramos los grupos que queremos:


Grupo, Texto
1. `<TR>`
2. `<FONT face=verdana,sans-serif size=2>1/3/2013 3:49:05 PM`
3.  
4. `EMS COSTA RICA`
5.  
6. `Depósito/ Recogida`
7.  
8. `Registro del envío`
9. `</TR><TR><TD bgColor=#dcdcdc><FONT face=verdana,sans-serif size=2>1/4/2013 7:09:31 AM`
10. `<TD bgColor=#dcdcdc>OFICINA HEREDIA`
11. `<TD bgColor=#dcdcdc>Llegada a la oficina distribuidora`
12. `<TD bgColor=#dcdcdc>`
13. `</TR><TR>`
14. `<FONT face=verdana,sans-serif size=2>1/4/2013 2:08:00 PM`
15.  
16. `OFICINA HEREDIA`

...

De acuerdo a esa separación los que nos interesan son el 2 y el 14:

	curl -s  --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' 

Que nos da de salida:

	<FONT face=verdana,sans-serif size=2>1/3/2013 3:49:05 PM<FONT face=verdana,sans-serif size=2>1/4/2013 2:08:00 PM

Y ahora podemos aplicar de nuevo el awk pero utilizando como separador de campos >\|< (< o >), que nos da los siguientes grupos:

Grupo, Texto
1.  	 
2. `FONT face=verdana,sans-serif size=2`
3. `1/3/2013 3:49:05 PM`
4. `FONT face=verdana,sans-serif size=2`
5. `1/4/2013 2:08:00 PM`
6. 
.  

	curl -s  --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp \| grep "Entregado a" \| awk -F "</*TD>" '/.*/ {print $2 $14}' \|  \| awk -F ">\|<" '/.*/ {print "EE123456," $3 "," $5}'

Dándonos la salida final que buscamos: EE123456, 1/3/2013 3:49:05 PM, 1/4/2013 2:08:00 PM

Sin embargo, eso fue sólo de una guía ¿cómo hacemos eso mismo para cien o mil?

## Replicando con awk

Suponiendo que tenemos la lista de guías en un simple archivo de texto **guias.csv** queremos replicar la línea de comando anterior con la que extrajimos las fechas de una guía pero para cada una de las guías de la lista:

	EE123451
	EE123452
	EE123453
	EE123454
	EE123455

Con el mismo **awk** podemos generar un script para luego ejecutar todas las consultas de una sola vez, indicando como separador de campos el retorno del final de cada línea \n y con el único cuidado de escapar correctamente las comillas dobles y simples[^3]:

	awk -F "\n" '/.*/ {print "curl -s  --data \"Envio_id=" $1 "&Consultar=Consultar\" https://www.correos.go.cr/rastreo/tracking2.asp | grep \"Entregado a\" | awk -F \"</*TD>\" '\''/.*/ {print $2 $14}'\'' | awk -F \">|<\" '\''/.*/ {print \"" $1 ",\" $3 \",\" $5 }'\'' "}' guias.csv > consulta.sh

Obteniendo como salida un archivo **consulta.sh** con el contenido:

	curl -s  --data "Envio_id=EE123451&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' | awk -F ">|<" '/.*/ {print "EE123451," $3 "," $5 }' 
	curl -s  --data "Envio_id=EE123452&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' | awk -F ">|<" '/.*/ {print "EE123452," $3 "," $5 }' 
	curl -s  --data "Envio_id=EE123453&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' | awk -F ">|<" '/.*/ {print "EE123453," $3 "," $5 }' 
	curl -s  --data "Envio_id=EE123454&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' | awk -F ">|<" '/.*/ {print "EE123454," $3 "," $5 }' 
	curl -s  --data "Envio_id=EE123455&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp | grep "Entregado a" | awk -F "</*TD>" '/.*/ {print $2 $14}' | awk -F ">|<" '/.*/ {print "EE123455," $3 "," $5 }' 

Que tras ejecutarlo nos va a dar la información que estábamos buscando:

	EE123451,1/3/2013 3:49:05 PM,1/4/2013 3:13:31 PM
	EE123452,1/4/2013 4:09:25 PM,1/5/2013 4:38:18 PM
	EE123453,1/5/2013 3:23:42 PM,1/6/2013 2:33:36 PM
	EE123454,1/6/2013 2:19:12 PM,1/7/2013 4:07:29 PM
	EE123455,1/7/2013 5:34:17 PM,1/8/2013 3:35:15 PM


 
---
[^1]: Aunque tomamos como ejemplo de courier a Correos de Costa Rica la solución general aplica para cualquier website que permita la consulta de una guía via HTTP.
[^2]: En Linux ese sería el comportamiento, en OSX no (y probablemente en BSDs). El funcionamiento de grep en OSX es diferente por la codificación de los caracteres en el texto fuente (el HTML), que en el caso de esta consulta Web la codificación retornada es iso-8859-1 y el uso de las tildes justo en la línea que contienen los datos de interés hace que grep no encuentre apartir de ese punto en la línea el Entregado a. Esto es un caso interesante digno de una entrada más profunda en el blog, pero por ahora vamos nada más a darle solución, salvándonos el día otro programa del que disponemos en la línea de comando: iconv. Con este podemos convertir la codificación original a UTF-8 con lo que grep (en OSX) encontrará la hilera buscada sin poblemas: curl -s --data "Envio_id=EE123456&Consultar=Consultar" https://www.correos.go.cr/rastreo/tracking2.asp \| iconv -f iso-8859-1 -t UTF-8 \| grep "Entregado a"  
[^3]: Un detalle interesante es la manera en que debemos escapar las comillas simples: ‘'’.
