---
layout: post
title:  "Memoization: recordando su historia"
date:   2014-07-27
categories: history programming
---

# 1943

No es una exageración decir que la historia de las funciones memo inicia en la Segunda Guerra Mundial, en uno de los lugares más famosos en la historia de lo secreto: el fascinante Bletchley Park. Este, agrupó mentes brillantes con el objetivo de decifrar las comunicaciones encriptadas alemanas y une su historia en 1943 con un recién graduado del entrenamiento de 6 semanas en criptografía llamado Donald Michie.

Michie se incorpora a la fuerza militar británica al ser reclutado para trabajar en el proyecto Testery, cuyo objetivo era romper el sistema de cifrado alemán llamado “Tunny”, que utilizaba una máquina aún más sofisticada que la famosa Enigma.

Luego de la guerra, estudió en el Balliol College en Oxford en donde recibió su doctorado en genética de mamíferos en 1953 y de ahí en adelante no hubo un año en que no realizara al menos una publicación sobre biología, informática o inteligencia artificial.

# 1968

Lamentablemente, en ocasiones damos por un hecho la existencia de conocimientos que encontramos triviales, olvidando que hubo un tiempo en que no existían y que una o más personas inventaron o descubrieron nuevos conceptos, nuevas ideas, que luego fueron incorporadas en la siempre creciente red del conocimiento humano.

En cuanto a la historia de memoization, es mucho más glamorosa de lo que me había imaginado. Según Wikipedia, el término fue acuñado por Michie en 1968 cuando hizo la publicación de un artículo titulado *“Memo” Functions and Machine Learning* (Funciones “Memo” y aprendizaje de máquinas), nada menos que en la revista ¡Nature en 1968! [^1]

En mi no-oficial traducción, el párrafo de introducción dice:

“Sería útil si las computadoras pudieran aprender de la experiencia y por ello automáticamente mejorar la eficiencia de sus programas durante la ejecución. Una simple pero efectiva manera de aprender de forma bruta puede ser provista dentro del marco de un lenguaje de programación”

# Memoization

La publicación en Nature, es más interesante que un simple cache para funciones, que es la implementación común en los lenguajes y que se puede ver como un simple mapa que almacena los valores de salida correspondientes a los parámetros de entrada para retornarlos sin recalcular sus valores. Por ejemplo en Clojure la función no requiere ningún cambio para obtener los beneficios, sólo se cambia la definición:

```clojure
(defn f [x] (cálculo-costoso x)) ;; sin memoization

(defn-memo f [x] (cálculo-costoso x)) ;; con memoization
```


Pero este tipo de implementación no es el más importante para Michie, sino un uso más sofisticado: “…Pero el uso de funciones memo puede ser extendido más allá de este limitado objetivo, de manera que se le confieran los poderes de aprendizaje por generalización y razonamiento inductivo.”

Michie presenta un ejemplo sobre la simulación de un vagón que sostiene un poste vertical e intenta como objetivo mantenerlo en posición vertical. Lamentablemente hay pocos detalles sobre la implementación, sólo menciona que la solución fue programada por el método de las “cajas”[^2].

Fuera del caso trivial que se implementa en algunos lenguajes, se pueden encontrar algunos artículos de robótica de los últimos años en donde se hace referencia a Michie y alguno de sus algoritmos, lamentablemente la mayoría parecen estar detrás de *“paywalls”*[^3].

Algunos enlaces relacionados:

- En 1991, Peter Norvig[^4] publicó un artículo titulado en el que aplica la técnica de memoization a parsers: Techniques for automatic memoization with applications to context-free parsing[^5].
- [El Instituto Turing](https://en.wikipedia.org/wiki/The_Turing_Institute)
- [Recordando a Donald Michie](http://vanemden.wordpress.com/2009/06/12/i-remember-donald-michie-1923-2007)
- [Una entrevista a Michie (video en Youtube)](https://www.youtube.com/watch?v=wR08vi64GDQ)

---
[^1]: D. Michie. Memo functions and machine learning. Nature, 218, 6 1968.
[^2]: Donald Michie and R. Chambers Boxes: An experiment in adaptive control. Machine Intelligence 2, 1968.
[^3]: Sistema que impide a los usuarios de la Internet accesar el contenido de una página sin una subscripción pagada.
[^4]: Peter Norvig es una de las personalidades en el ámbito de la inteligencia artificial, LISP, análisis lingüístico (entre otros). Tiene un sitio Web con excelente contenido que vale la pena visitar: [https://norvig.com](https://norvig.com).
[^5]: P. Norvig. Techniques for automatic memoization with applications to context-free parsing. Computational Linguistics, 17(1):91–98, 03 1991.
