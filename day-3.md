<p align="center">
<img src="img/logo.svg" width="400"  alt="" title="Defsquare"/>
</p>


# Day 3

<!-- TOC -->
* [Day 3](#day-3)
  * [4.b Domain - Schema control & Entity](#4b-domain---schema-control--entity)
  * [4.b Domain - Command & Query](#4b-domain---command--query)
  * [4.c Domain - Control function inputs and outputs](#4c-domain---control-function-inputs-and-outputs)
  * [4.d Domain - Define repository behavior](#4d-domain---define-repository-behavior)
  * [5.a Application service - Implementing](#5a-application-service---implementing)
  * [6.a API Spec - Implementing](#6a-api-spec---implementing)
  * [6.b API Spec - Documentation](#6b-api-spec---documentation)
  * [7.a Repository - Datasource](#7a-repository---datasource)
  * [7.b Repository - JDBC Datasource](#7b-repository---jdbc-datasource)
  * [8.d Testing - Implementing test](#8d-testing---implementing-test)
  * [8.e Testing - Hooks](#8e-testing---hooks)
  * [9.c Observe - Implementing log system](#9c-observe---implementing-log-system)
<!-- TOC -->

## 4.b Domain - Schema control & Entity
- clojure.spec
- Schema
- Malli

```clojure
; Malli offers data generative capability
(require '[malli.generator :as mg])

(mg/generate Client)
;=> {:id "Fr502r0fl4H", :name "84KJrITHskM38a1XSl74dxtOmw", :age 30619}

; Refining your schema to be more accurate
(def Client [:map
             [:id [:re #"client-\d{5}"]]
             [:name [:re #"\w*{20}"]]
             [:age [:int {:min 1 :max 120}]]])

(mg/generate Client)
;=> {:id "client-08237", :name "NhnD1H2ldFQ9", :age 12}
```

Implementing Entity
```clojure
; Utility function to log errors if data is not valid
(defn valid? [schema data]
      (if-let [errors (me/humanize (m/explain schema data))]
              (do (log/error "Invalid data : " errors) false)
              true))
; Then use it in prepost map
(defn my-fn [data]
      {:pre  [(valid? InputSchema data)]
       :post [#(valid? OutputSchema %)]}
      ;do stuff
      )

; Define a builder for your entity as well
(defn ->entity [data]
      ;do stuff
      )
```

## 4.b Domain - Command & Query
```clojure
(def DoSomething [:map
                  [:issued-by string?]
                  [:command-name :do-something]
                  [:payload [:map [:criteria string?]]]])

(defn ->do-something [data] #_body)
(defn my-fn [data]
      {:pre [(valid? DoSomething data)]}
      #_body)
; You could use defmulti as a polymorphic mechanism to handle command/query
(defmulti handle (fn [command-or-query] (or (:command-name command-or-query) (:query-name command-or-query))))
(defmethod handle :do-something [command]                  
           #_body)
```

## 4.c Domain - Control function inputs and outputs
```clojure
(def CreateClientCommand [:map [:username string?] [:age int?]]) ;; Use prepost map which contains a collection of predicate to validate both function input and output

(defn create-client [command]
      {:pre  [(m/validate CreateClientCommand command)]
       :post [#(m/validate Client %)]}
      {:id   (str (:username command) "-" (rand-int 10000))
       :name (:username command)
       :age  (:age command)})

(create-client {:username "dpanza" :age 45})
;=> {...}

(create-client {:username "dpanza" :age "45"})
;=> Assert failed: (m/validate CreateClientCommand command)
```

## 4.d Domain - Define repository behavior
```clojure
; Defined by domain as a PORT
(defprotocol EntityRepository
             (find-by-id [repo entity-id] "Returns an entity")
             (store-entity [repo entity] "Persists the entity"))

; Then domain logic calls an abstraction
(defn handle [command ^EntityRepository repository]
      (let [entity (find-by-id repository (:entity-id command))]
           #_body))

; Implemented by infrastructure as an ADAPTER
(defrecord InmemEntityRepository [state-atom]
           EntityRepository
           (find-by-id [repo entity-id] #_body)
           (store-entity [repo entity] #_body))

(defstate inmem-repo
          :start (->InmemEntityRepository (atom []))
          :stop nil)
```

## 5.a Application service - Implementing
```clojure
(ns com.defsquare.training.app.controller.entity-api
    (:require
      [com.defsquare.training.domain.my-entity :refer [my-domain-operation]]
      [com.defsquare.training.app.repository.my-repo :refer [inmem-repo]]))

(def entity-router
  ["/api"
   ["/entity"
    {:post
     {:handler
      (fn [request]
          (let [{:keys [username age]} (:query-params request)
                entity (my-domain-operation inmem-repo {:username username :age age})]
               {:status 200 :body entity}))}}]])
```

## 6.a API Spec - Implementing
```clojure
(require '[reitit.coercion.malli :as malli-coercion])

(def app
  (ring/ring-handler
    (ring/router
      ["/api"
       ["/entity/:id" {:post {:parameters {:body  [:map #_...]
                                           :path  [:map #_...]
                                           :query [:map #_...]}
                              :responses  {200 {:body [:map #_...]}}
                              :handler    (fn [request] #_...)}}]]
      {:data {:coercion malli-coercion/coercion}})))
```

## 6.b API Spec - Documentation
```clojure
(require '[reitit.swagger :as swagger]
         '[reitit.swagger-ui :as swagger-ui])

(def app
  (ring/ring-handler
    (ring/router
      [["/api"
        ["/entity/:id" {:post {:description "create an entity"
                               :parameters  #_{...}
                               :responses   #_{...}
                               :handler     #_...}}]]
       ["" {:no-doc true}
        ["/swagger.json" {:get (swagger/create-swagger-handler)}]
        ["/api-docs/*" {:get (swagger-ui/create-swagger-ui-handler)}]]]
      {:data #_{...}})))

; A swagger specification is available at http://localhost:8080/swagger.json   
; A swagger ui is available at http://localhost:8080/api-docs/index.html 
```

## 7.a Repository - Datasource
https://github.com/seancorfield/next-jdbc
```clojure
;; in your deps.edn
com.github.seancorfield/next.jdbc {:mvn/version "1.3.883"}
com.h2database/h2 {:mvn/version "1.4.199"}

;; in your clojure NS
(require '[next.jdbc :as jdbc])

(def db-spec {:dbtype   "h2"
              :dbname   "training"
              :host     "localhost"
              :port     9092
              :user     "sa"
              :password "pwd"})
```

## 7.b Repository - JDBC Datasource
```clojure
(defstate datasource
          :start (jdbc/get-datasource db-spec)
          :stop nil)

(jdbc/execute! datasource ["
create table users (
 id    varchar(32) primary key,
 name  varchar(32),
 email varchar(255))"])

(jdbc/execute! datasource ["insert into users(id,name,email) values(?, ?, ?)"
                           "dpanza-1", "David PANZA", "david@defsquare.com"])

(jdbc/execute! datasource ["select * from users"])
```

## 8.d Testing - Implementing test
```clojure
(ns defsquare.training-test  
    (:require 
      [clojure.test :refer [deftest testing is]] 
      [defsquare.training :as sut]))

(deftest my-fn-test
         (testing "My fn returning something"
                  (is (= (sut/my-fn 1) 1) "should return 1")))

; `testing` is a macro to add context (as documentation) and could be nested; `is` as a generic assertions taking a predicate
; 2nd parameter to `is` is the message to return if assertion fail
(comment  
  (clojure.test/run-tests) ; Finally, a function to execute all test in NS
)
```

## 8.e Testing - Hooks
```clojure
(use-fixtures :once                      ; Execute this once for NS
              (fn [tests]
                  (prn "Start components") ; Mount/Start your components (DB, server)
                  (tests)                  ; Execute tests
                  (prn "Stop components")  ; Dismount/Stops all components
                  ))

(use-fixtures :each      ; Execute this for each test
              (fn [test] (prn "Populate a context data")
                  (test)                           ; Execute tests
                  (prn "Clean data generated")     ; Dismount/Stops all components
                  ))
```

## 9.c Observe - Implementing log system
https://cambium-clojure.github.io/about.html 

Add dependencies in deps.edn
```clojure
{:deps  {cambium/cambium.core              {:mvn/version "1.1.1"}
         cambium/cambium.codec-simple      {:mvn/version "1.0.0"}
         cambium/cambium.logback.core      {:mvn/version "0.4.5"}}}
```

In your resource folder, create a logback config file in `resources/logback.xml`
```xml
<configuration>
    <appender name="STDOUT" class="ch.qos.logback.core.ConsoleAppender">
        <encoder>
            <pattern>%d{HH:mm:ss.SSS} [%thread] %-5level %logger{36} - %msg { %mdc }%n</pattern>
        </encoder>
    </appender>
    
    <root level="info">
        <appender-ref ref="STDOUT" />
    </root>
</configuration>
```

In a clojure namespace 
```clojure
; Use log function info/trace/debug/warn
(log/info {:foo "bar"} "Start creating entity")
; Persists a context as structured data for log contained in body
(log/with-logging-context {:request {:verb "get" :uri "/api/entity/123"}}
                          (log/info {:foo "bar"} "Start creating entity"))
```

[Previous Day](day-2.md) | [Go to home](README.md) | [Next Day](day-4.md)