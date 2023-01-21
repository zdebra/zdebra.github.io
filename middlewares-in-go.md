---
title: "Middlewares In Go"
date: 2017-08-06T18:14:50+02:00
draft: true
---

# HTTP middleware v Go

Pojmy jako Aspect Oriented Programming (AOP), Cross-cutting concern či Middleware - to všechno sem patří. Velmi stručně, v kontextu webových aplikací, se moduly, jemiž musí HTTP požadavek během své životnosti projít. Hello World webová aplikace v Go vypadá takto:

```go
package main

import (
    "fmt"
    "net/http"
)

func handler(w http.ResponseWriter, r *http.Request) {
    fmt.Fprintf(w, "Hi there, I love %s!", r.URL.Path[1:])
}

func main() {
    http.HandleFunc("/", handler)
    http.ListenAndServe(":8080", nil)
}
```

Teď chcete každý request zalogovat, ověřit uživatele a jeho role a nastavit nějaké HTTP hlavičky. Pro to máme interface `http.Handler`, který vyžaduje mít definovanou jedinou funkci a to `ServeHTTP(ResponseWriter, *Request)`. Trvalo mi docela dlouho, než jsem přišel na ten nejlepší způsob: Adapter (nebo také Decorator) pattern. Napsal o tom Mat Rayer [tady](https://medium.com/@matryer/the-http-handler-wrapper-technique-in-golang-updated-bc7fbcffa702).


