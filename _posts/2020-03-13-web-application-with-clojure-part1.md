---
layout: post
title: Web application with clojure
date: '2020-03-13'
categories: posts
published: false
tags: [clojure, pedestal service]
---
Building self hosted comments service with clojure. I will be using pedestal-service library to  build web application. This will be like hands on session to create an application with pedestal-service library.

### Create clojure application using leiningen and pedestal-service
Use `leiningen` to create new pedestal service. This will pull templates that required for basic web application
`lein new pedestal-service comments`

Below is list of files after generation.

```bash
.
├── Capstanfile
├── Dockerfile
├── README.md
├── comments.iml
├── config
│   └── logback.xml
├── logs
│   └── comments-2020-03-13.0.log
├── project.clj
├── resources
├── src
│   └── comments
│       ├── server.clj
│       └── service.clj
├── target
│   ├── classes
│   │   └── META-INF
│   │       └── maven
│   │           └── comments
│   │               └── comments
│   │                   └── pom.properties
│   └── stale
│       └── leiningen.core.classpath.extract-native-dependencies
└── test
    └── comments
        └── service_test.clj

14 directories, 12 files
```

### Run application
`lein run`

Default port is 8080, open `http://localhost:8080/` to see "Hello world"
Main entry point for application is `src/comments/server.clj`
routes and port are defined in `src/comments/service.clj`

### Rendering html
I am using `hiccup` library to render HTML. This will be acting as templating and rendering library. Hiccup is a library for turning a Clojure data structure into a string of HTML.

Add `[hiccup "1.0.5"]` into your project.clj

Learn more about `hiccup` [here](https://github.com/weavejester/hiccup/wiki/Syntax)


## Connecting database
