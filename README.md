<div align="center">

![Flow](https://raw.githubusercontent.com/alexedwards/flow/assets/flow-sm.png)
        
[![Go Reference](https://pkg.go.dev/badge/github.com/alexedwards/flow.svg)](https://pkg.go.dev/github.com/alexedwards/flow) [![Go Report Card](https://goreportcard.com/badge/github.com/alexedwards/flow)](https://goreportcard.com/report/github.com/alexedwards/flow) [![MIT](https://img.shields.io/github/license/alexedwards/flow)](https://img.shields.io/github/license/alexedwards/flow) ![Code size](https://img.shields.io/github/languages/code-size/alexedwards/flow)

A delightfully simple, readable, and tiny HTTP router for Go web applications.
</div>

---

Flow is a simple but powerful router. It packs in a bunch of features that you'll probably like:

* Use **named parameters**, **wildcards** and (optionally) **regexp patterns** in your routes.
* Create route **groups which use different middleware** (a bit like chi).
* **Customizable handlers** for `404 Not Found` and `405 Method Not Allowed` responses.
* **Automatic handling** of `OPTIONS` and `HEAD` requests.
* Works with `http.Handler`, `http.HandlerFunc`, and standard Go middleware.
* Tiny, readable, codebase (~160 lines of code).

---

### Installation

```
$ go get github.com/alexedwards/flow@latest
```

### Basic example

```go
package main

import (
    "fmt"
    "log"
    "net/http"

    "github.com/alexedwards/flow"
)

func main() {
    // Initialize a new router.
    mux := flow.New()

    // Create a route which dispatches `GET /greet/:name` requests to the
    // `greet` handler. This : character is used to define a named parameter in
    // the URL path, so this route will be used for requests like `GET /greet/alice` 
    // and `GET /greet/bob`.
    mux.HandleFunc("/greet/:name", greet, "GET")

    err := http.ListenAndServe(":2323", mux)
    log.Fatal(err)
}

func greet(w http.ResponseWriter, r *http.Request) {
    // Use flow.Param() to retrieve the value of the named parameter from the
    // request context.
    name := flow.Param(r.Context(), "name")

    fmt.Fprintf(w, "Hello %s", name)
}
```

### Kitchen-sink example

```go
mux := flow.New()

// The Use() method can be used to register middleware. Middleware declared at
// the top level will used on all routes (including error handlers and OPTIONS
// responses).
mux.Use(exampleMiddleware)

// Routes can use multiple HTTP methods.
mux.HandleFunc("/profile/:name", exampleHandlerFunc, "GET", "POST")

// Optionally, regular expressions can be used to enforce a specific pattern
// for a named parameter.
mux.HandleFunc("/profile/:name/:age|^[0-9]{1,3}$", exampleHandlerFunc, "GET")

// The wildcard ... can be used to match the remainder of a request path.
// Notice that HTTP methods are also optional (if not provided, all HTTP
// methods will match the route).
mux.Handle("/static/...", exampleHandler)

// You can create route 'groups'.
mux.Group(func(mux *flow.Mux) {
    // Middleware declared within in the group will only be used on the routes
    // in the group.
    mux.Use(exampleMiddleware)

    mux.HandleFunc("/admin", exampleHandler, "GET")

    // Groups can be nested.
    mux.Group(func(mux *flow.Mux) {
        mux.Use(exampleMiddleware)

        mux.HandleFunc("/admin/passwords", exampleHandlerFunc, "GET")
    })
})
```

### Notes

* Routes are matched in the order that they are declared.
* Trailing slashes are significant (`/profile/:id` and `/profile/:id/` are not the same).
* An `Allow` header is automatically set for all `OPTIONS` and `405 Method Not Allowed` responses (including when using custom handlers). 

### Contributing

Bug fixes and documentation improvements are very welcome! For feature additions or behavioral changes, please open an issue to discuss the change before submitting a PR.

### Thanks

The pattern matching logic for Flow was heavily inspired by [matryer/way](https://github.com/matryer/way).