---
layout: post
title: "El curioso caso de una botnet y el Colegio de Ciencias Económicas de Costa Rica"
date:   2022-09-24
---

---

En Agosto del 2020, Guardicore, una empresa israelita de ciber seguridad publicó un estudio sobre una nueva y sofisticada botnet[^1] que denominaron FritzFrog.

La empresa fue luego [adquirida por nada menos que Akamai](https://www.akamai.com/newsroom/press-release/akamai-to-acquire-guardicore-to-extend-its-zero-trust-solutions-to-help-stop-ransomware1) (600 millones de dólares) por lo que el sitio web original ya no existe. Akamai tiene una [página simplificada](https://www.akamai.com/blog/security/fritzfrog-p2p) sobre el estudio pero la publicación con el análisis original se puede leer gracias a archive.org en [FritzFrog: A New Generation of Peer-to-Peer Botnets](https://web.archive.org/web/20200819131202/https://www.guardicore.com/2020/08/fritzfrog-p2p-botnet-infects-ssh-servers/).

Al día siguiente, después de terminar de leer el análisis tuve la curiosidad/preocupación de saber si alguna máquina en Costa Rica estaba infectada. Dichosamente los investigadores publicaron junto al análisis un [repositorio en GitHub](https://github.com/guardicore/labs_campaigns/tree/master/FritzFrog) que incluye listas de IPs[^2] que detectaron relacionados con la actividad de las infecciones.

Busqué y descargué listas con bloques de IPs por país, escribí un pequeño programa para encontrar IPs de Costa Rica y en las listas del análisis encontré una dirección[^3]. 

Probando el IP, ssh respondió y el puerto 1234 estaba abierto. También tenía puerto de Web abierto y al cargarlo en el navegador obtuve un sitio del Colegio de Ciencias Económicas de Costa Rica, un formulario de ingreso a una aplicación Web.

Usando una consulta de IP a ASN[^4] obtuve que el IP pertenece a un bloque del ICE y que me permitió encontrar un contacto telefónico y una dirección para reportar casos de Spam pero no tuve ninguna respuesta. 

Revisando el sitio del Colegio llamé a una extensión telefónica que se indicaba relacionada con el departamento de informática pero nadie contestó. Escribí por correo, identificándome con mi nombre completo con la esperanza de que no lo descartaran como spam o como alguien tratando de hacer algún fraude. Tampoco obtuve respuesta. 

Hace 2 años de esto.

¿Fue una falsa alarma la inclusión del IP en la lista del análisis publicado?

¿Fue ese servidor parte de una botnet sin que nadie se diera cuenta?

¿Lo supo alguien?

 ¯\\_(ツ)_/¯


---
[^1]: [¿Qué es una botnet?](https://www.cloudflare.com/es-es/learning/ddos/what-is-a-ddos-botnet)
[^2]: [¿Qué es una dirección IP?](https://help.gnome.org/users/gnome-help/stable/net-what-is-ip-address.html.es)
[^3]: 201.195.86.14
[^4]: [Autonomous System Numbers](https://es.wikipedia.org/wiki/Sistema_aut%C3%B3nomo)
