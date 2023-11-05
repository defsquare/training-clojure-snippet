<p align="center">
<img src="img/logo.svg" width="400"  alt="" title="Defsquare"/>
</p>

# Day 4

<!-- TOC -->
* [Day 4](#day-4)
  * [2.a Interop - Importing JAVA](#2a-interop---importing-java)
  * [2.b Interop - Instantiate JAVA class](#2b-interop---instantiate-java-class)
  * [2.c Interop - Cast and JAVA interface implementation](#2c-interop---cast-and-java-interface-implementation)
  * [2.d Interop - Call clojure function from JAVA](#2d-interop---call-clojure-function-from-java)
  * [3.b Web - Web project setup](#3b-web---web-project-setup)
  * [3.b Web - Shadow-cljs](#3b-web---shadow-cljs)
  * [4.a HTML - Hiccup](#4a-html---hiccup)
  * [5.a Component - Reagent](#5a-component---reagent)
  * [5.b Component - Implementation](#5b-component---implementation)
  * [5.c Component - State](#5c-component---state)
  * [5.d Component - Component typologie](#5d-component---component-typologie)
<!-- TOC -->

## 2.a Interop - Importing JAVA
```clojure
; A function exists to import java in NS
(import java.time.Instant)

; The `ns` macro supports imports too: 
(ns defsquare.training
    (:require #_...)
    (:import (java.time LocalDate Instant)))

; All java.lang.* classes are imported automatically in NS
(String/valueOf 1)
(Integer/valueOf "1")

; Inner classes are separated from their outer class by a dollar sign `$`
(ns defsquare.training
    (:require #_...)
    (:import (java.util AbstractMap$SimpleEntry)))

(AbstractMap$SimpleEntry. "foo" "bar")
```

## 2.b Interop - Instantiate JAVA class
```clojure
; Java classes are instantiated using the `new` special form
(new java.util.Date)

; But clojure reader provides a syntactic sugar as well
(java.util.Date.)

; Arguments are provided next to constructor
(java.util.Date. 123 3 27)

; Static methods are called using the separator `/`
(java.time.Instant/now)

; or the special `.` form
(. java.time.Instant now)

; Invoking methods on instance use the special form `.`
(. (java.time.Instant/now) toString)

; The common way is method next to the dot
(.toString (java.time.Instant/now))

; Chaining invocation by using special form `..` 
(.. (java.time.Instant/now) getEpochSecond toString)

; If you need to call several methods on the same instance, use `doto` macro
(doto (java.util.ArrayList.)
      (.add "foo")
      (.add "bar")
      (.add "kix"))
```

## 2.c Interop - Cast and JAVA interface implementation
```clojure
; Sometimes, you need to pass from clojure class to java class
(int-array [1 2 3])
(long-array [1 2 3])
(boolean-array [true true])
(char-array [\1 \2 \3])

; Implement Java interface by using `reify` special form
(let [person (reify java.util.concurrent.Callable
                    (call [this] "yes?"))]
     (.call person))
;=> yes

; `reify` returns a java class instance. Compiler will generate a class that implements the interface and instantiate it

```

Java Interface
```java
//package com.defsquare.training;

public interface Moveable extends Node {
    String turnRight();
    String turnLeft();
}
```

```clojure
; Could be implemented by clojure with `reify` macro
(reify Moveable
       (turnRight [this] (Position. ))
       (turnRight [this] (Position. )))

; Or record
(defrecord MyPosition [x y]
           Moveable
           (turnRight [this] (str (inc x) "-" y))
           (turnLeft [this] (str (dec x) "-" y)))
```

## 2.d Interop - Call clojure function from JAVA

```java
public class Foo {
    // Call clojure from java
    public Integer sum(Integer a, Integer b) {
        IFn plus = Clojure.var("clojure.core", "+");
        return (Integer) plus.invoke(a, b);
    }

    // You could call your own clojure function from your NS
    public Collection<String> map(Collection<String> data) {
        IFn map = Clojure.var("clojure.core", "map");
        IFn transformFn = Clojure.var("com.defsquare.training", "transform-fn");
        return (Collection<String>) map.invoke(transformFn, data);
    }
}
```

## 3.b Web - Web project setup
https://github.com/defsquare/training-clojure-cljs 

```clojure
{:paths ["src/main"]
 :deps  {org.clojure/clojure       {:mvn/version "1.11.1"}
         org.clojure/clojurescript {:mvn/version "1.11.60"}
         hiccup/hiccup             {:mvn/version "2.0.0-RC1"}
         reagent/reagent           {:mvn/version "1.2.0"}}
 :aliases
 {:shadow-cljs {:extra-deps {thheller/shadow-cljs {:mvn/version "2.25.7"}}
                :main-opts  ["-m" "shadow.cljs.devtools.cli"]}}}
```

## 3.b Web - Shadow-cljs
```clojure
{:deps   true ; use tools.deps file as dependencies manager
 :builds {:app {:asset-path "/js" ; indicates to browser js files location
                :target     :browser ; selected target
                :output-dir "resources/public/js" ; where to generate js files
                :modules    {:main {:init-fn training-cljs.core/init!}}
                :devtools   {:http-root "resources/public" ;mount a server
                             :http-port 3000}}}
 :nrepl  {:port 9000}} ; configuration for nRepl
```

```shell
clj -M:shadow-cljs watch app
```

## 4.a HTML - Hiccup
https://github.com/weavejester/hiccup 

```clojure
; HTML markup represented as vector
[:div]                    ; first element is HTML tag as keyword
[:div "Hello World!"]     ; next elements are nested
[:div [:p "Hello World!"]]

; Attributes are represented as map placed at 2nd index
[:a {:href "http://defsquare.com"} "click here!"]

; Can be manipulated with clojure core function like data
(conj [:a {:href "http://defsquare.com"}] "click here!")
;=> [:a {:href "http://defsquare.com"} "click here!"]

(->> ["Hello" "World"]
     (map (fn [s] [:p s]))
     (into [:div]))
;=> [:div [:p "Hello"] [:p "World"]]

```

## 5.a Component - Reagent
https://github.com/reagent-project/reagent

## 5.b Component - Implementation
```clojure
; Define a component like a function and use Hiccup syntax inside
(defn sub-component [s] [:div s])

(sub-component "Hello World!") ; generate a vector with elements
[sub-component "Hello World!"] ; create a reagent component 

; Nested components invocation
(defn main-component []
      [:div
       [:p "Here's title"]
       [:div [sub-component "Hello World!"]]])

[main-component]
;=> [:div
;     [:p "Here's title"]
;     [:div [:div "Hello World!"]]])
```

## 5.c Component - State
```clojure
; Reagent provide its own implementation of atom to manage component state
(defonce counter (r/atom 0))

; Any component which dereference reagent atom will be re-rendered
(defn sub-component [s]
      [:div {:on-click #(swap! counter inc)} (str s @counter)])

; Atom can be included in component creation
(defn sub-component [s]
      (let [state (r/atom 0)] ; initiates state at first when created (once)
           (fn [s]                ; provide render function
               [:div {:on-click #(swap! state inc)} (str s @state)])))


; Structure of reagent component
(defn sub-component [s]       ;<--- input component data 
      (let [state (r/atom 0)] ;<--- ratom as component state
           (fn [s]            ;<--- render function like f(data)=view
               [:div {:on-click #(swap! state inc)} (str s @state)])))

; If input data or deferenced atom change, the render function is called and generate new HTML
```

## 5.d Component - Component typologie
```clojure
; Way to create component

; 1. A simple function (form 1)
(defn sub-component [s] [:div s]) ; just a function which returns hiccup

; 2. A function returning a function (form 2)
(defn sub-component [s]
      (let [state (r/atom 0)]  ; setup and  local state
           (fn [s]             ; inner render function
               [:div {:on-click #(swap! state inc)} (str s @state)])))

; 3. A function returning a class (form 3)
(defn sub-component [s]
      (let [state (r/atom 0)]
           (r/create-class
             {:component-did-mount  (fn [this] #_...)
              :component-did-update (fn [this old-args] #_...)
              :reagent-render       (fn [s] [:div {:on-click #(swap! state inc)} (str s @state)])})))
```

[Previous Day](day-3.md) | [Go to home](README.md)

