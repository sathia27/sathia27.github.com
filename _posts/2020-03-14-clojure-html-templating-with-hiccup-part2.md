---
layout: post
title: Writing web application in Clojure - Part 2
date: '2020-03-13'
categories: posts
published: false
tags: [clojure, pedestal service, hiccup]
---

### Create clojure application using leiningen and pedestal-service
Use `leiningen` to create new pedestal service. This will pull templates that required for basic web application
`lein new pedestal-service comments`

### Have public resource file
By default, clojure have it's resource folder, where you can add all your public and stylesheet and JavaScript files.

In `comments.service`, you can change path in `::http/resource-path`
Changing it as below

```clojure
::http/resource-path "/public"
```
And created `application.css` as below:

```bash
├── resources
│   └── public
│       └── css
│           └── application.css
```

### Creating routes for route path.

In `comments.service`

```clojure
(def routes #{["/" :get (conj common-interceptors `new-comment)]})
```

### Creating basic HTML template using hiccup
Creating `new-page` as defined in above routes.

```clojure
(defn new-comment
  [request]
  (ring-resp/response (hiccup/html5
                        [:head
                          (hiccup/include-css "https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css")
                          (hiccup/include-css "css/application.css")]
                        [:div {:class "container"}
                          [:h1 "Add comments"]
                          [:form ]
                         ])))
```
I have included application.css from local public folder and also including bootstrap css from cdn as well.

### Create a form

```html
<form>
  <div class="form-group">
    <label for="postUrl">Url</label>
    <input type="text" class="form-control" id="postUrl" placeholder="Enter post url">
  </div>
  <div class="form-group">
    <textarea class="form-control" rows="3" placeholder="Enter comments"></textarea>
  </div>
  <button type="submit" class="btn btn-default">Submit</button>
</form>
```

Above form can be written in `hiccup` as below:

```clojure
(defn new-comment
  [request]
  (ring-resp/response (layout/application
                        "New comment"
                        [:h1 "New comment"]
                        [:form
                          [:div {:class "form-group"}
                            [:label "URL"]
                            [:input {:type "text" :class "form-control" :id "postUrl" :placeholder "Enter post url"}]]
                          [:div {:class "form-group"}
                            [:textarea {:class "form-control" :placeholder "Enter commemts"}]]
                          [:submit-button {:class "btn btn-default"} "Submit"]])))
```

### How to re-use layout?

Creating new layout file which includes application layout.

```clojure
(ns comments.views.layout
  (:require [hiccup.page :as hiccup]))

(defn application
  [title & content]
  (hiccup/html5
    [:head
     [:title title]
     (hiccup/include-css "https://stackpath.bootstrapcdn.com/bootstrap/3.4.1/css/bootstrap.min.css")
     (hiccup/include-css "css/application.css")]
    [:body
     [:div {:class "container"} content]])
  )
```
And using application layout in `new-page` and it can be refactored as below code.

```clojure
(defn new-comment
  [request]
  (ring-resp/response (layout/application
                        "New comment"
                        [:h1 "New comment"]
                        [:form {:action "/comments" :method "POST"}
                          [:div {:class "form-group"}
                            [:label "URL"]
                            [:input {:type "text" :class "form-control" :id "postUrl" :placeholder "Enter post url"}]]
                          [:div {:class "form-group"}
                            [:textarea {:class "form-control" :placeholder "Enter commemts"}]]
                          [:button {:type "submit" :class "btn btn-default"} "Submit"]])))
```

Not only building form, you can create any html elements using `hiccup`

Go through other syntax of `hiccup` [here](http://weavejester.github.io/hiccup/hiccup.page.html)
