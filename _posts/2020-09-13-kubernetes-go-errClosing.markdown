---
layout: post
title: "Extraño comentario en Kubernetes"
date:   2020-09-13
---

---

¿Qué hacer en una apacible tarde de Domingo en pandemia?
Lo que hace todo el mundo por supuesto: leer código...

Leyendo fuentes en Go me puse a explorar el [repositorio](https://github.com/kubernetes/kubernetes/) de [Kubernetes](https://kubernetes.io/) y en uno de los archivos encontré un comentario curioso: 

{% highlight go%}

var (
	// ErrPipeListenerClosed is returned for pipe operations on listeners that have been closed.
	// This error should match net.errClosing since docker takes a dependency on its text.
	ErrPipeListenerClosed = errors.New("use of closed network connection")

	errPipeWriteClosed = errors.New("pipe has been closed for write")
)

{% endhighlight %}

La advertencia *"This error should match net.errClosing since docker takes a dependency on its text."*[^1] sorprende en la primera lectura, ¿por qué una hilera que describe un error debe ser exactamente así porque otro proyecto (Docker) depende a su vez de ella? ¿por qué esa dependencia? 

Algo olió extraño.

## Docker

En el [repositorio de Docker](https://github.com/docker/docker-ce) encontramos [6 veces la hilera *"use of closed network connection"*](https://github.com/docker/docker-ce/search?q=%22use+of+closed+network+connection%22&unscoped_q=%22use+of+closed+network+connection%22) en 2 tipos de casos:

- Retornando error en [client_io_windows.go](https://github.com/docker/docker-ce/blob/98ddba151eebbff519cc86342443c3a09f8ce334/components/engine/libcontainerd/remote/client_io_windows.go):

{% highlight go%}
return 0, errors.New("use of closed network connection")

{% endhighlight %}

- En comparaciones como una de las condiciones de *if*s en el manejo de errores (!):

{% highlight go%}
strings.Contains(err.Error(), "use of closed network connection")
{% endhighlight %}

Este último tipo de comparación, aparece en Docker en 5 archivos distintos:
- [engine/api/server/server.go](https://github.com/docker/docker-ce/blob/d73611ce7f0e62337faab4e06db2f3e6ed048436/components/engine/api/server/server.go)
- [engine/cmd/dockerd/metrics.go](https://github.com/docker/docker-ce/blob/95f98b104bc1ee4c5e04be0e1e6418090f9537ef/components/engine/cmd/dockerd/metrics.go)
- [engine/daemon/metrics_unix.go](https://github.com/docker/docker-ce/blob/95f98b104bc1ee4c5e04be0e1e6418090f9537ef/components/engine/daemon/metrics_unix.go)
- [engine/integration/internal/requirement/requirement.go](https://github.com/docker/docker-ce/blob/c5aa6c97d6f63fe42e93bd55fe622726755217a5/components/engine/integration/internal/requirement/requirement.go)
- [engine/integration-cli/requirements_test.go](https://github.com/docker/docker-ce/blob/c88e552800adfe359051dede4b56ca12d54bba83/components/engine/integration-cli/requirements_test.go)



## ¿Cuándo?
Centrándonos en una ocurrencia, le seguimos la pista en [engine/api/server/server.go](https://github.com/docker/docker-ce/blob/d73611ce7f0e62337faab4e06db2f3e6ed048436/components/engine/api/server/server.go#L86), encontrando el *commit* en el que se introduce la línea [^3]:

{% highlight diff %}
@@ -1561,7 +1578,15 @@ func ServeApi(job *engine.Job) engine.Status {
                                chErrors <- err
                                return
                        }
-                       chErrors <- srv.Serve()
+                       job.Eng.OnShutdown(func() {
+                               if err := srv.Close(); err != nil {
+                                       log.Error(err)
+                               }
+                       })
+                       if err = srv.Serve(); err != nil && strings.Contains(err.Error(), "use of closed network connection") {
+                               err = nil
+                       }
+                       chErrors <- err
                }()
        }

{% endhighlight %}

En el commit del [20 de marzo del 2015](https://github.com/docker/docker-ce/commit/449ca606a0a82fe3a00c1eb80dd3b4a6ca86793b) se introduce la revisión del error: si el texto del error es *"use of closed network connection"* se **descarta el error** (ignorándose con *err = nil*).


## ¿Por qué?

¿Por qué distribuir en múltiples funciones y luego como consecuencia en múltiples proyectos una hilera específica de error y no actuar dependiendo del **type** del error? 
El cambio sugiere que el tipo de error que puede ocurrir en esos puntos tiene diversas causas y al menos una de ellas (que ocurre con ese texto en particular) se puede ignorar. 

Tras una corta búsqueda, encontramos la causa documentada en el [issue #4373](https://github.com/golang/go/issues/4373) en el repositorio de Go:
**"net: errClosing not exported"** del 11 de noviembre del **2012**. El [tercer comentario](https://github.com/golang/go/issues/4373#issuecomment-66071976) confirma la suposición anterior:

<blockquote>
Based on my experience, there seems to be quite a few patterns with goroutines in
servers where you end up with 2 goroutines associated with the same Conn and where
developers (correctly, or not) deem it necessary that both goroutines close that Conn in
some error paths.
In this case, some people want to distinguish between errClosing, which is an expected
error and can be ignored, and any other error from Close (my favourite is EBADF), which
should probably be fatal to the program (i.e. the program should panic when it happens).
The general vibe seems to be that one should simply ignore the error from Close instead
of trying to distinguish between expected and unexpected errors, but this still doesn't
sit completely well with me...
</blockquote>

Todo se debe a que en ese momento el *package* net de Go retornaba

{% highlight go %}
var errClosing = errors.New("use of closed network connection")
{% endhighlight %}

Pero sin exportar un **type** en el *package*, de forma que nadie podía comparar si el error era ***E**rrClosing*.

Casi ocho años después, [encontramos la solución](https://go-review.googlesource.com/c/go/+/250357/3/src/net/net.go#84) [^2] en el commit del 24 de agosto del **2020** en [net/net.go](https://github.com/golang/go/blob/master/src/net/net.go): 

{% highlight diff %}
@@ -81,6 +81,7 @@ package net
 import (
        "context"
        "errors"
+       "internal/poll"
        "io"
        "os"
        "sync"
@@ -632,6 +633,17 @@ func (e *DNSError) Timeout() bool { return e.IsTimeout }
 // error and return a DNSError for which Temporary returns false.
 func (e *DNSError) Temporary() bool { return e.IsTimeout || e.IsTemporary }
 
+// errClosed exists just so that the docs for ErrClosed don't mention
+// the internal package poll.
+var errClosed = poll.ErrNetClosing
+
+// ErrClosed is the error returned by an I/O call on a network
+// connection that has already been closed, or that is closed by
+// another goroutine before the I/O is completed. This may be wrapped
+// in another error, and should normally be tested using
+// errors.Is(err, net.ErrClosed).
+var ErrClosed = errClosed
+
{% endhighlight %}

Y en [internal/poll/fd.go](https://github.com/golang/go/blob/86dbeefe1f2770daad3c8d8b46a8b7f21b2c69e1/src/internal/poll/fd.go#L20):

{% highlight go %}
// ErrNetClosing is returned when a network descriptor is used after
// it has been closed. Keep this string consistent because of issue
// #4373: since historically programs have not been able to detect
// this error, they look for the string.
var ErrNetClosing = errors.New("use of closed network connection")
{% endhighlight %}



----

[^1]: *Este error debe coincidir con net.errClosing ya que Docker depende de este texto*
[^2]: En el repositorio de Docker: git diff bbc96d66b4 449ca606a0 components/engine/api/server/server.go
[^3]: En el repositorio de Docker: git diff e9ad52e46d^ src/net/net.go