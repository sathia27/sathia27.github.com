---
layout: post
title: Go private fields in Struct and JSON
date: '2021-08-13'
categories: posts
published: true
tags: [golang, programming, basics, golang basics, structs]
---

There was a recent experience when I was playing around with Go-lang's struct and JSON marshal. Take a look at below example


```go
package main

import (
   "fmt"
   "encoding/json"
)

type Trip struct {
   tripId string
   tripType string
}

func main() {
    trip := Trip{tripType: "air"}
    fmt.Println(trip)

    tripJson, err := json.Marshal(trip)
    fmt.Println(err)
    fmt.Println(string(tripJson))
}
```

Below is output

```bash
{ air}
<nil>
{}
```

Did you notice the above code? Within the same package printing `trip` returns the data (`{ air}`) correctly but `tripJson` returns empty (`{}`) with no error. It's because the `json` package will silently ignore private fields in your struct.

### Un-exported / private fields.

Any method or variables starting with a small case letter will be considered as private / Unexported fields. That's not how "privacy" works in Go

In the above case, `json` is a separate package, so the `json` package will not have access for setting/getting private / Unexported fields.

### Fixing above struct

```go
package main

import (
   "fmt"
   "encoding/json"
)

type Trip struct {
   TripId string
   TripType string
}

func main() {
    trip := Trip{TripType: "air"}

    tripJson, _ := json.Marshal(trip)
    fmt.Println(string(tripJson))
}
```

Below is output

```bash
{"TripId":"","TripType":"air"}
```

### JSON module calling private fields with Struct tags

```go
package main

import (
   "fmt"
   "encoding/json"
)

type Trip struct {
   tripId string `json:"trip_id"`
   tripType string `json:"trip_type"`
}

func main() {
    trip := Trip{tripType: "air"}
    fmt.Println(trip)

    tripJson, err := json.Marshal(trip)
    fmt.Println(err)
    fmt.Println(string(tripJson))
}
```


With the above code, `go vet` will throw warnings for using exported fields when it is tagged with a json tag.

```bash
➜go vet trip/trip.go
trip/trip.go:9:4: struct field tripId has json tag but is not exported
trip/trip.go:10:4: struct field tripType has json tag but is not exported
```

It's always safe to use json tags even when keys are the same, this helps to avoid mistakes to some extent. When invoking `vet` would show warnings when you are using private variables with `json` tag.

Note: Go compiler still will not throw error for above scenario.

```bash
➜go run trip/trip.go
{ air}
<nil>
{}
```