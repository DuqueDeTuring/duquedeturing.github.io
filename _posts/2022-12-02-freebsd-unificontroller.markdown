---
layout: post
title: "Evitando el error en FreeBSD al instalar el controlador de UniFi desde ports"
date:   2022-12-02
---

---


El controlador de Unifi está disponible en los ports de FreeBSD (13.1), sin embargo, tratando de instalarlo en un _jail_ en mi homelab desde ports fallaba al tratar de instalar una dependencia: Perl (y con un error relacionado a dtrace).

*sigh*

En lugar de tratar de arreglar el port de Perl (tal vez otro día...) la alternativa fue muy simple: bastó con instalar Perl con `pkg install perl5` y luego hacer de nuevo el make del port `unifi7`.

Instalamos Perl con:

{% highlight bash %}
# pkg install perl5
{% endhighlight %}
 

Y luego intentamos de nuevo con el port del _Unifi controller_:

{% highlight bash %}
# make install clean BATCH=yes 
{% endhighlight %}

Terminando con:

{% highlight bash %}
# service unifi start
{% endhighlight %}

...y listo! 

