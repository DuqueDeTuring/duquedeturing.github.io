---
layout: post
title:  "'for' en Clojure, no es un ciclo"
date:   2011-12-28
categories: clojure programming
---

En "¿Dónde está mi ciclo?", capítulo 2 de [*Programming Clojure*](http://pragprog.com/book/shcloj2/programming-clojure), se aclara que "for" en [Clojure](https://clojure.org/) no es un ciclo sino una operación sobre listas (*"list comprehension"*).

Aunque, podría verse un ciclo como una operación sobre listas, después de años de usar "for" en múltiples lenguajes para ciclos, es un ejercicio mental cambiar la reacción inmediata y automática al ver la palabra "for" en un código Clojure.

Para asimilar mejor su uso, podemos reescribir un ejemplo de "for" del libro pero usando otras operaciones sobre listas pero con el mismo resultado:

```clojure
(use '[clojure.java.io :only (reader)])                                                                                                                                                                                                   

(defn non-blank? [line] (if (re-find #"\S" line) true false))                                                                                                                                                                             

(defn clojure-source? [file] (.endsWith (.toString file) ".clj"))                                                                                                                                                                         

(defn clojure-loc2 [base-file]                                                                                                                                                                                                            
  (reduce + 
          (map 
              #(with-open [rdr (reader %)] (count (filter non-blank? (line-seq rdr)))) 
              (filter clojure-source? (file-seq base-file)))))  
```

[Gist](https://gist.github.com/1529386)
