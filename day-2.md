<p align="center">
<img src="img/logo.svg" width="400"  alt="" title="Defsquare"/>
</p>

# Day 2

<!-- TOC -->
* [Day 2](#day-2)
  * [2.a Standard - Indentation](#2a-standard---indentation)
  * [2.b Standard - Namespace](#2b-standard---namespace)
  * [2.c Standard - Naming](#2c-standard---naming)
  * [2.d Standard - Function](#2d-standard---function)
  * [2.e Standard - Threading](#2e-standard---threading)
  * [3. Eco-system - Clojure eco-system](#3-eco-system---clojure-eco-system)
  * [4.a Url Shortener](#4a-url-shortener)
  * [5.a Backend - Initiate a clojure app project](#5a-backend---initiate-a-clojure-app-project)
  * [5.c Backend - State management](#5c-backend---state-management)
  * [6.a API - Web server](#6a-api---web-server)
  * [6.b API - Manual routing](#6b-api---manual-routing)
  * [6.c API - Router](#6c-api---router)
  * [6.d API - Encode/Decode](#6d-api---encodedecode)
  * [7.a Persistence - Inmem repository](#7a-persistence---inmem-repository)
<!-- TOC -->

## 2.a Standard - Indentation

Body indentation
```clojure
;; good
(when something
  (something-else))

(with-out-str
  (println "Hello, ")
  (println "world!"))

;; bad - four spaces
(when something
    (something-else))

;; bad - one space
(with-out-str
 (println "Hello, ")
 (println "world!"))
```

Function Argument alignment
```clojure
;; good
(filter even?
        (range 1 10))

;; bad - argument aligned with function name (one space indent)
(filter even?
        (range 1 10))

;; bad - two space indent
(filter even?
        (range 1 10))
```

Function Argument Indentation
```clojure
;; good
(filter
  even?
  (range 1 10))

;; bad - two-space indent
(filter
  even?
  (range 1 10))

```
Binding Alignment
```clojure
;; good
(let [thing1 "some stuff"
      thing2 "other stuff"]
     (foo thing1 thing2))

;; bad
(let [thing1 "some stuff"
      thing2 "other stuff"]
     (foo thing1 thing2))
```

Gather Trailing Parentheses
```clojure
;; good; single line
(when something
      (something-else))

;; bad; distinct lines
(when something
      (something-else)
      )
```

Empty lines between Top-Level Forms
```clojure

;; good
(def x ...)

(defn foo ...)

;; bad
(def x ...)
(defn foo ...)

;; Exception for grouping of related def’s together

;; good
(def min-rows 10)
(def max-rows 20)
(def min-cols 15)
(def max-cols 30)

```

No blank lines within definition forms
```clojure

;; okay - the line break delimits a cond pair
(defn fibo-iter
      ([n] (fibo-iter 0 1 n))
      ([curr nxt n]
       (cond
         (zero? n)
         curr

         :else
         (recur nxt (+ curr nxt) (dec n)))))

;; bad
(defn fibo-iter
      ([n] (fibo-iter 0 1 n))

      ([curr nxt n]
       (cond
         (zero? n) curr

         :else (recur nxt (+ curr nxt) (dec n)))))
```

## 2.b Standard - Namespace
Avoid single-segment namespaces (conflict risks)
```clojure
;; good
(ns example.ns)

;; bad
(ns example)
```

## 2.c Standard - Naming
Use lisp-case for function and variable names
```clojure
;; good
(def some-var ...)
(defn some-fun ...)

;; bad
(def someVar ...)
(defn somefun ...)
(def some_fun ...)
```

Use CapitalCase for protocols, records, structs and types
```clojure

;; good
(defprotocol Moveable ...)
(defrecord Car ...)

;; bad
(defprotocol moveable ...)
(defrecord car ...)
```

The names of predicate functions (returning a boolean) should end in a question mark
```clojure
;; good
(defn palindrome? ...)

;; bad
(defn palindrome-p ...) ; Common Lisp style
(defn is-palindrome ...) ; Java style
```

Use earmuffs for things intended for rebinding
```clojure
;; good
(def ^:dynamic *a* 10)

;; bad
(def ^:dynamic a 10)
```

Don’t use a special notation for constants; everything is assumed a constant
```clojure
;; good
(def max-size 10)

;; bad
(def MAX-SIZE 10) ; Java style
(def +max-size+ 10) ; Common Lisp style, global constant
(def *max-size* 10) ; Common Lisp style, global variable
```

Use ‘_’ for destructuing targets and formal argument names whose value will be ignored by the body form
```clojure
;; good
(let [[a b _ c] [1 2 3 4]]
  (println a b c))

(dotimes [_ 3]
  (println "Hello!"))

;; bad
(let [[a b c d] [1 2 3 4]]
  (println a b d))

(dotimes [i 3]
  (println "Hello!"))
```

For documenting-purpose, you can explicit a name for unused arguments or maps by prepending a ‘_’
```clojure
;; good
(defn myfun1 [context _]
 (assoc context :foo "bar"))

(defn myfun2 [context {:keys [id]}]
 (assoc context :user-id id))

;; better
(defn myfun1 [context _user]
 (assoc context :foo "bar"))

(defn myfun2 [context {:keys [id] :as _user}]
 (assoc context :user-id id))
```

## 2.d Standard - Function

Separate function name and argument from function body
```clojure
;; good
(defn foo  "My foo documentation"
  [x]
  (bar x))

;; good
(defn foo [x]
  (bar x))

;; bad
(defn foo
  [x] (bar x))
```

Indent each arity form of a function definition vertically aligned with its parameters
```clojure
;; good
(defn foo
  "I have two arities."
  ([x]
   (foo x 1))
  ([x y]
   (+ x y)))

;; bad - extra indentation
(defn foo
  "I have two arities."
  ([x]
    (foo x 1))
  ([x y]
    (+ x y)))
```

Sort arity of a function from fewest to most arguments
```clojure
;; good - it's easy to scan for the nth arity
(defn foo
  "I have two arities."
  ([x]
   (foo x 1))
  ([x y]
   (+ x y)))

;; bad 
(defn foo
  ([x] 1)
  ([x y z] (foo x (foo y z)))
  ([x y] (+ x y))
  ([w x y z & more] (reduce foo (foo w (foo x (foo y z))) more)))
```

Don’t define ns vars inside functions
```clojure
;; very bad
(defn foo []
  (def x 5)
  #_body)
```

Don’t shadow clojure.core names with local bindings
```clojure
;; bad - clojure.core/map must be fully qualified inside the function
(defn foo [map]
  #_body)
```

## 2.e Standard - Threading
Don’t shadow clojure.core names with local bindings
```clojure
;; good
(-> [1 2 3]
    reverse
    (conj 4)
    prn)

;; not as good
(prn (conj (reverse [1 2 3])
           4))
```

## 3. Eco-system - Clojure eco-system

- HTTP server: [http-kit-server](https://github.com/http-kit/http-kit), <font size="4">[ring](https://github.com/ring-clojure/ring)</font> 
- State management: [component](https://github.com/stuartsierra/component), [integrant](https://github.com/weavejester/integrant), <font size="4">[mount](https://github.com/tolitius/mount)</font>
- Data Driven Schemas: <font size="4">[malli](https://github.com/metosin/malli)</font> , [clojure.spec](https://clojure.org/guides/spec), [Schemas](https://github.com/plumatic/schema)
- Data Transformation: <font size="4">[meander](https://github.com/noprompt/meander)</font>, [clara-rules](http://www.clara-rules.org)
- Rule engine: <font size="4">[clara-rules](http://www.clara-rules.org)</font>
- Test lifecycle management: <font size="4">[kaocha](https://github.com/lambdaisland/kaocha)</font>, [test-runner](https://github.com/cognitect-labs/test-runner) 
- Clojure Project lifecycle: [Leiningen](https://leiningen.org), [boot](https://github.com/boot-clj/boot),  <font size="4">[tools.deps](https://clojure.org/guides/deps_and_cli)</font>
- Configuration Management: <font size="4">[aero](https://github.com/juxt/aero)</font>, [environ](https://github.com/weavejester/environ), [config](https://github.com/yogthos/config)
- Web Router: <font size="4">[reitit](https://github.com/metosin/reitit)</font>, [yada](https://github.com/juxt/yada)
- Time: <font size="4">[tick](https://github.com/juxt/tick)</font>, [clj-time](https://github.com/clj-time/clj-time)
- JDBC-based access: <font size="4">[nextjdbc](https://github.com/seancorfield/next-jdbc)</font>, [honeysql](https://github.com/seancorfield/honeysql), [hugSQL](https://www.hugsql.org)
- Kafka: <font size="4">[java-kafka](https://docs.confluent.io/kafka-clients/java/current/overview.html)</font>, [jackdaw](https://github.com/FundingCircle/jackdaw)
- Async programming: <font size="4">[core.async](https://github.com/clojure/core.async)</font>, [manifold](https://github.com/clj-commons/manifold)
- Testing library: [midje](https://github.com/marick/Midje), <font size="4">[testit](https://github.com/metosin/testit)</font>


## 4.a Url Shortener
```gherkin
Feature: Url shortened creation
  Scenario: A user creates his shortened url
    Given a user "dpanza"
    When "dpanza" creates a short url for "https://clojure.org/"
    Then the short url is stored
    And a short url is given to user

Feature: Manage url shortened
  Scenario: A user lists all the urls created
    Given a user "dpanza"
    And existing shortened urls
      | user   | link                 | shortened | visits |
      | dpanza | https://clojure.org/ | clj       | 12     |
    When "dpanza" gets his urls shortened    
    Then the returned list of url shortened return "1" results
    Then the list contains link for "https://clojure.org/" shortened to "clj" visited by "12" users

Feature: Redirect visitors
  Scenario: A visitor clicks a shortened url
    Given existing shortened urls
      | user   | link                 | shortened | visits |
      | dpanza | https://clojure.org/ | clj       | 12     |
    When a visitor clicks on a shortened url "clj"
    Then the visitor is redirected to "https://clojure.org/"
    And the number of visits for the shortened url "clj" of "dpanza" is incremented
```

## 5.a Backend - Initiate a clojure app project
```shell
# Create a project using tools-deps
clojure -Ttools install com.github.seancorfield/clj-new '{:git/tag "v1.2.404"}' :as clj-new

clojure -Tclj-new app :name defsquare/training

ls –l 
 CHANGELOG.md
 LICENSE
 README.md
 build.clj
 deps.edn
 doc
 pom.xml
 resources
 src
 test
```

## 5.c Backend - State management
https://github.com/tolitius/mount

in your deps.edn file, add mount
```clojure
{:deps {mount/mount {:mvn/version "0.1.17"}}}
```

in a clojure namespace
```clojure
(defstate my-state          
          :start (do (prn "starting") {:status :run})
          :stop (do (prn "stopping") {:status :stopped}))

(defn -main [& args] (mount.core/start))

(comment
  (main) 
  (mount.core/start)
  (mount.core/stop))
```

## 6.a API - Web server
https://github.com/ring-clojure/ring

in your deps.edn file, add mount
```clojure
{:deps {ring/ring-core          {:mvn/version "1.10.0"}
        ring/ring-jetty-adapter {:mvn/version "1.10.0"}}}

;; in a clojure namespace

(defn handler [_request]
      {:status  200
       :headers {"Content-Type" "text/html"}
       :body    "Hello World"})

(defstate server          
          :start (run-jetty handler {:port 8080 :join? false})
          :stop (.stop server))
```

## 6.b API - Manual routing
```clojure
(defmulti router (fn [{:keys [uri request-method] :as _request}]
                     [request-method uri]))

(defmethod router [:get "/api/entity"]
           [_request]
           {:status  200
            :headers {"Content-Type" "text/html"}
            :body    "This is an entity!"})

(defmethod router :default
           [{:keys [uri]}]
           {:status  404
            :headers {"Content-Type" "text/html"}
            :body    (str "Request '" uri "' not found")})

(defstate server
          :start (run-jetty router {:port  8080 :join? false})
          :stop (.stop server))
```
## 6.c API - Router
https://github.com/metosin/reitit

in your deps.edn file
```clojure
{:deps {metosin/reitit {:mvn/version "0.7.0-alpha5"}}}
```

in a clojure namespace
```clojure
(def app
  (ring/ring-handler
    (ring/router      ["/api"
                       ["/entity" {:get (fn [_request]
                                            {:status  200
                                             :headers {"Content-Type" "text/html"}
                                             :body    "This is an entity!"})}]])))

(comment  
  (app {:request-method :get, :uri "/api/entity"}))
```

## 6.d API - Encode/Decode
```clojure
(:require 
  [muuntaja.core :as m]
  [reitit.ring.coercion :as coercion]
  [reitit.ring.middleware.muuntaja :as muuntaja]
  [reitit.ring.middleware.parameters :as parameters])

(def app
  (ring/ring-handler
    (ring/router
      ["/api"
       ["/entity/:id"
        {:post         
         {:handler
          (fn [request]
              (let [{:keys [body-params path-params query-params]} request]
                   {:status 200 :body   ""}))}}]]
      {:data {:muuntaja   m/instance
              :middleware [muuntaja/format-middleware
                           coercion/coerce-exceptions-middleware
                           coercion/coerce-request-middleware
                           coercion/coerce-response-middleware
                           parameters/parameters-middleware]}})))
```

## 7.a Persistence - Inmem repository
```clojure
(def initial-state {:users {}})
(def db (atom initial-state))

@db
;; => {:users {}} 

(swap! db assoc-in [:users :dpanza] {:foo "bar"})
;; => {:users {:dpanza {:foo "bar"}}}

@db
;; => {:users {:dpanza {:foo "bar"}}}

(reset! db initial-state)
;; => {:users {}} 

@db
;; => {:users {}}
```

[Previous Day](day-1.md) | [Go to home](README.md) | [Next Day](day-3.md)