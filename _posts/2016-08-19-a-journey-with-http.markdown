---
published: true
title: A Journey with Http
layout: post
---

I was in a journey with http for last few days,  was started from one of my old programs which was written in my college days.
When I have just gone through my old github repositories, one of those programs was came to my attention, especially a couple of lines.

```c
if(listen(welcomeSocket,5)==0)
   printf("Listening on 7000\n");

```
It was a tcp server. I have'nt written any socket programs in C since my college days. But still it looks like very familiar to me. I could recollect where I have used the same kind of code within a fraction of second. It may be familiar for you,  if you are a web developer.

If you cant recollect where it have been used, take a look at these code snippets.

```golang
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

```js
var express = require('express');
var app = express();

app.get('/', function (req, res) {
  res.send('Hello World!');
});

app.listen(3000, function () {
  console.log('Example app listening on port 3000!');
});
```
