<p align="center">
  <img src="img/logo.svg" width="400"  alt="" title="Defsquare"/>
</p>

# Day 1

<!-- TOC -->
* [Day 1](#day-1)
  * [2.b VSCode - Create clojure project](#2b-vscode---create-clojure-project)
  * [5.a Data structure - List](#5a-data-structure---list)
  * [5.a Data structure - Vector](#5a-data-structure---vector)
  * [5.a Data structure - Map](#5a-data-structure---map)
  * [5.b Data structure - Set](#5b-data-structure---set)
  * [5.c Data structure - Sequence](#5c-data-structure---sequence)
  * [5.d Data structure - Keyword](#5d-data-structure---keyword)
  * [5.e Data structure - Symbol](#5e-data-structure---symbol)
  * [5.f Data structure - Record](#5f-data-structure---record)
  * [5.g Data structure - Set](#5g-data-structure---set)
  * [6.a Core Function - Arithmetic operation](#6a-core-function---arithmetic-operation)
  * [6.b Core Function - Collection](#6b-core-function---collection)
  * [6.c Core Function - Sequence](#6c-core-function---sequence)
  * [6.d Core Function - Predicate](#6d-core-function---predicate)
  * [6.e Core Function - Logical](#6e-core-function---logical)
  * [6.f Core Function - Control flow](#6f-core-function---control-flow)
  * [6.h Core Function - Function](#6h-core-function---function)
  * [6.i Core Function - Namespace](#6i-core-function---namespace)
  * [6.j Core Function - Concurrency and state](#6j-core-function---concurrency-and-state)
  * [6.k Core Function - Dispatching & polymorphism](#6k-core-function---dispatching--polymorphism)
  * [7 Practice - Let's practice !](#7-practice---lets-practice-)
<!-- TOC -->


## 2.b VSCode - Create clojure project
Via tools.deps
```shell
brew install clojure
clojure -Ttools install com.github.seancorfield/clj-new '{:git/tag "v1.2.404"}' :as clj-new
clojure -Tclj-new app :name defsquare/training-day-1
```

Via Leiningen
```shell
brew install clojure leiningen
lein new defsquare-training-day-1
```


## 5.a Data structure - List
```clojure
(1 2 3)
(fn [] true)
```

## 5.a Data structure - Vector
```clojure
[1 2 3]
["foo" "bar"]
```


## 5.a Data structure - Map
```clojure
{"key1" "value1"}
```

## 5.b Data structure - Set
```clojure
#{1 2 3}
#{"foo" "bar"}
```


## 5.c Data structure - Sequence
```clojure
(seq)
(seq [1 2 3])
(seq #{1 2 3})
```

## 5.d Data structure - Keyword
```clojure
:xxx
:my-keyword
::my-fully-qualified-keyword
:namespace/my-keyword
```

## 5.e Data structure - Symbol
```clojure
'…
'my-var
'(fn [x] (str x))
```

## 5.f Data structure - Record
```clojure
(defrecord Name [] #_behaviors )
(defrecord position [x y] Moveable
           (move-forward [_] (->position (inc x) y)))
```

## 5.g Data structure - Set
```clojure
#{1 2 3}
#{"foo" "bar"}
```

## 6.a Core Function - Arithmetic operation
```clojure
(+ 2 1)
(- 2 1)
(* 2 1)
(/ 2 1)
```

## 6.b Core Function - Collection
```clojure
(first [1 2]) ;Takes first and second element
;=> 1
(second [1 2])
;=> 2

(rest [1 2 3]) ;Takes the rest items from collection, excluding the first one
;=> [2 3]

(count [1 2 3]) ;Returns the numbers of elements in collection
;=> 3
(empty? []) ;Returns true if any element in collection
;=> true

(conj [1 2] 3) ;Adds parameters (from 2nd) to the 1st parameter
;=> [1 2 3]
(cons 1 [2 3]) ;Adds the element to the 1st position in collection
;=> (1 2 3)

(concat [1 2] [3]) ;Appends the members of one sequence to the end of another
;=> (1 2 3)
(into [1 2] #{3}) ;Combines element and convert collection to the 1st one
;=> [1 2 3]
```


## 6.c Core Function - Sequence
```clojure
(seq [1 2]), (seq #{3 4}) ;Convert a collection into Sequence
;=> (1 2), (3 4)

(map str [1 2]) ;Iterates over elements by applying function on each
;=> ("1" "2")
(filter odd? [1 2]) ;Removes elements that which does not respect the predicate
;=> (1)
(reduce (fn [acc itm] (conj acc (str "the-" itm))) #{} [1 2]) ;Iterates over elements and return another data structure
;=> #{"the-1" "the-2"}

(take 2 [:a :b :c]) ;Returns the first N elements in seq
;=> (:a :b)
(drop 2 [:a :b :c]) ;Removes N elements from the seq
;=> (:c)
```

## 6.d Core Function - Predicate
```clojure
;‘=‘, ‘<‘, ‘>’, ‘<=‘, ‘>=‘ Comparison operators which return true
(= 1 1)
;=> true

;‘even?’, ‘odd?’ Returns true if even or odd parameter
(even? 2)
;=> true
(odd? 2)
;=> false

;‘nil?’ Checks if value is equal to something
(nil? true) ;=> false
(nil? nil) ;=> true

;‘true?’, ’false?’ Checks the Boolean value 
(true? true) ;=> true
(true? false) ;=> false

;‘identical?’ Checks if two objects have the same reference (equality)
(identical? {:foo "bar"} {:foo "bar"}) ;=> false
(identical? :foo :foo) ;=> true
```

## 6.e Core Function - Logical
```clojure
(and 1 2) ;Returns the last element
;=> 2
(and (= 1 2) 2) ;but checks logical value for form element and stop iterating at first false value
;=> false

(or 1 2) ;Returns the first element
;=> 1; 
(or (= 1 2) 2) ;but evaluate form element a return the first logical true evaluation
;=> 2

(not 1) ;Checks if argument is logical false

((complement empty?) [1]) ;Returns a function for the opposite logical value of a predicate
;=> true

```
## 6.f Core Function - Control flow
```clojure
(when (odd? 1) "odd!") ;Checks 1st form logical value and return the last form
;=> "odd!"

(if (odd? 2) "odd!" "even!") ;Checks 1st form logical value, returns then or else
;=> "even!"

(loop [x [1 2]]
   (if (empty? x) x (recur (x)))) ;Tail recursion mechanism over bindings
;=> ()

(cond 
  (= 1 0) "foo"
  :else :kix)             ;Conditional execution composed by predicates
;=> :kix

(for [x [1 2] y [:a :b]]
  [x y])                  ;Cartesian product trough bindings
;=> ([1 :a] [1 :b] [2 :a] [2 :b])

(do (prn "hello") true) ;Evaluates all form (presumably for side effects)

(doseq [x [1 2]]
       (prn x))          ;Iterates over bindings (presumably for side effects)
;=> nil

(-> 10 (- 2) str) ;Threading execution as a pipeline where previous are inserted as 1st argument of the next form
;=> "8"

(->> 10 (- 2) str) ;Same but previous are inserted as last argument of the next
;=> "-8"

(cond-> 10
        (true? true) (- 2)
        :always str) ;Conditional threading evaluation 
;=> "8"
(cond->>)
```

## 6.h Core Function - Function
```clojure
(fn [x] (str x)) or #(str %) ;Defines anonymous function
;=> #reference

(defn stringify [x] (str x)) ;Defines a named function
(stringify 1)
;=> "1"

(defmacro foo [& body] …) ;Generates code by controlling the compilation
;=> #code

(let [x 1] x) ;Binds symbol to its value and make it available only for body
;=> 1
```

## 6.i Core Function - Namespace
```clojure
;‘def’, ‘defn’, ‘defmacro’ Defines symbols to their value in a namespace

(def my-var nil)
(defn my-function [] #_body )
(defmacro my-macro [] #_body )

(ns com.defsquare.training.clojure ;Declares and switches between namespaces
    (:require [clojure.string :as string :refer [blank?]]) ;import CLJ ns
    (:import (java.time LocalDate))) ;import JAVA classes
=> nil

(in-ns ‘foo)             ;Switches to another namespace in REPL
```

## 6.j Core Function - Concurrency and state
```clojure
(def state (atom {:foo "bar"})) ;Create a mutable value
;=> #’my-ns.state

(deref state) 		;Returns the value pointed by the reference
;=> {:foo "bar"}

(swap! state conj {:kix nil}) ;Apply function and args to the value 
;=> {:foo "bar}

(reset! state {:foo nil}) ;Reset value pointed by the reference
;=> {:foo nil}
```

## 6.k Core Function - Dispatching & polymorphism
```clojure
(defmulti http-code (fn [x] (:status x))) ;Define a value for dispatching
(defmethod http-code :ok [x] 200)         ;Register a hierarchy with a matcher
(defmethod http-code :incorrect [x] 400)  ;The matching uses ‘isA?’ predicate
(defmethod http-code :error [x] 500)


(http-code {:status :ok})
;=> 200
(http-code {:status incorrect})
;=> 400
(http-code {:status :foo})                ;If no dispatching function match, 
;=> Exception, no dispatch value           ;Raise a runtime exception

(defmethod http-code :default [x] 200)    ;A default function could be declared
(http-code {:status :foo})
;=> 200
```

## 7 Practice - Let's practice !
https://4clojure.oxal.org

- [Problem 19](https://4clojure.oxal.org/#/problem/19) – Last element
- [Problem 23](https://4clojure.oxal.org/#/problem/23) – Reverse a Sequence
- [Problem 24](https://4clojure.oxal.org/#/problem/24) – Sum it All Up
- [Problem 28](https://4clojure.oxal.org/#/problem/28) – Flatten a Sequence
- [Problem 44](https://4clojure.oxal.org/#/problem/44) – Rotate Sequence
- [Problem 54](https://4clojure.oxal.org/#/problem/54) – Partition a Sequence
- [Problem 70](https://4clojure.oxal.org/#/problem/70) – Word Sorting

[Go to home](README.md) | [Next Day](day-2.md)
