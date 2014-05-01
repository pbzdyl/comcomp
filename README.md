# comcomp

'Comcomp' helps integrate Compojure routes with Stuart Sierra's '[Component](https://github.com/stuartsierra/component)' library.

It provide `defroutes-with-deps` macro which you can use to describe 'route component'

```clojure
(defroutes-with-deps
                       AppComponent      ; <- Component name

                       [first-dep second-dep]        ; <- Dependency list.

                       (GET "/hello" [] (some-func first-dep))     ; <- Routes spec
                       (POST "/word" [] (some-second-func second)) ;You can use
                                                                   ;previously defined
                                                                   ;dependencies
```


## Usage:

```clojure
;Define routes component using compcomp macro defroutes-with-deps.
;Describe and use dependencies

(ns app
  (:require [com.stuartsierra.component :as comp]
            [comcomp.core :as comcomp]
            [compojure.handler :as handler]
            [ring.adapter.jetty :as rj])
  (:use compojure.core))

(comcomp/defroutes-with-deps
                       AppRoutes      ; <- Component name

                       [app-db]        ; <- Dependency list.

                       (GET "/word" [] (get-word app-db))    ; <- Routes spec
                       (GET "/count" [] (get-count app-db))) ; You can use previously defined
                                                             ; dependencies

;Macro will create AppRoutes record. It implements comp/Lifecycle and IRotesDescriber
;IRoutesDescriber defines get-routes, you can use this method to get described routes.

;Create routes component and provide dependencies.
(def system (comp/system-map  :app (comp/using (map->App {}) [:app-routes])
                              :app-db (comp/using ...)
                              :app-routes (comp/using (map->AppRoutes {}) ;<- create routes component
                                                       [:app-db]))) ;<- Provide dependency

;Use comcomp/get-routes to get described routes from routes component.

(defrecord App [app-routes]
  comp/Lifecycle
  (start [this]
    (assoc this :server
                  (rj/run-jetty (handler/site
                                  (routes (comcomp/get-routes app-routes))) ;<- Run app with described routes
                                  {:port 8080 :join? false})))
  (stop [this] (.stop (:server this)) this))

```

## License

Copyright © 2014

Distributed under the Eclipse Public License either version 1.0 or any later version.
