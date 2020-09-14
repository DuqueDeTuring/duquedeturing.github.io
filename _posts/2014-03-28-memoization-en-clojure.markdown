---
layout: post
title:  "Memoization en Clojure"
date:   2014-03-28
categories: clojure programming
---

Una técnica sumamente simple de utilizar y que a la vez puede brindar una mejora significativa en cómputos costosos (en funciones con transparencia referencial[^1]) es utilizar *memoization*[^2] (básicamente un caché de evaluación de una función).

Es más, de sencilla que es puede pasar desapercibida. En [Clojure](https://clojure.org) convertir una función normal a una función optimizada con *memoization* es trivial.

Si tenemos la función:
```clojure
(defn f [x] (* x 2))
```

Convertirla a una función con memoization basta con cambiar su definición a la macro defn-memo:

```clojure
(defn-memo f [x] (* x 2)
```

Difícil que sea más fácil ¿pero cómo funciona?

La macro defn-memo simplemente nos ahorra trabajo utilizando clojure.core/memoize, si no utilizáramos la macro, el código podría ser:

```clojure
(def f 
  (memoize
	(fn [x]
		(* x 2))))
```

*(¿Por qué def y no defn?[^3])*

Sencillo también, entonces veamos como funciona clojure.core/memoize.

Usando la versión de Clojure 1.6, el código fuente de clojure.core/memoize es:

```clojure
(defn memoize
  [f]
  (let [mem (atom {})]
    (fn [& args]
      (if-let [e (find @mem args)]
        (val e)
        (let [ret (apply f args)]
          (swap! mem assoc args ret)
          ret)))))
```

`memoize` genera un *closure* que va a mantener un mapa con la relación entre argumentos y la aplicación de la función.

En prosa, la descripción de la implementación sería la siguiente: se crea un identificador `mem` que hace referencia a un mapa que contiene el caché de valores de la función aplicada.

Cuando se evalúa la función, se revisa primero si en el mapa existen los argumentos, si existen se obtiene el valor asociado en el mapa, de lo contrario se aplica la función original `f` a los argumentos, guardando temporalmente el resultado en `ret` y luego, haciendo uso de la función `swap!` que permite “cambiar” valores en un `atom` (con identificador `mem` en este caso) se “registra” en el mapa (aplicando `assoc`) el resultado de la evaluación asociado a los argumentos.

Aunque es posible entender a ese nivel sin problemas la implementación, puede surgir la pregunta: ¿por qué utilizar mem como `atom`?

¿Cuál sería la alternativa?
```clojure
(defn memoize-sin-atom
  [f]
  (let [mem {}]
    (fn [& args]
      (if-let [e (mem args)]
        (val e)
        (let [ret (apply f args)]
          (assoc mem args ret)
          ret)))))
```

Si aplicamos esta función en el ejemplo original no vamos a encontrar diferencia…¿entonces para qué el `atom` y `swap!`?

La clave no está en la simple aplicación de `atom`, sino luego en el cambio del mapa con `swap!`.

`swap!` aplica una función (con argumentos si es necesario) sobre un `atom` de forma sincronizada, “atómica”. Es decir, gracias a la semántica de `swap!`, el código de `memoize` garantiza que en un ambiente multi-hilos, si más de un hilo trata de actualizar nuestro atom no se dará, por ejemplo, que dos hilos ocasionen la aplicación de nuestra costosa función `f` original, sino que la primera que ejecute el `swap!` será la que aplique la función y cualquier otro hilo que intente aplicar la función estará bloqueado reintentando hasta que haya terminado el cambio inicial, que claro, al evaluarse luego otro, podría usar ya el valor almacenado de la primera evaluación y almacenado en el atom `mem` (si fueran los mismos argumentos) y obtener el beneficio del valor preprocesado.


---
[^1]: Una función es referencialmente transparente si puede ser sustituida por su valor sin alterar el programa. En otras palabras, no se puede aplicar funciones con efectos secundarios (a no ser que estos sean triviales a nivel práctico).
[^2]: En términos técnicos, en lugar de buscar una aproximación en español, prefiero utilizar el original (aunque esté en otro idioma), aunque resulte en un spanglish, que dicho sea de paso parece inevitable en el campo de la informática.
[^3]: Si observamos la implementación, podemos ver que la aplicación de la función memoize da como resultado una función, de forma que la aplicación sea luego (f x). Si declaráramos nuestra nueva “memo” f con (defn f ...) en lugar de (def f ...) en realidad tendríamos una composición extra (g (f x)) obligándonos a hacer una aplicación más, por ejemplo: ((f 3)) en lugar de simplemente (f 3).

