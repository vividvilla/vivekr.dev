+++
title = "Why I made yet another sessions library in Go?"
description = ""
date = 2019-04-26
updated = 2019-04-26
draft = false
slug = "why-simplesessions-library-in-go"

[taxonomies]
tags = ["Go", "Github", "Golang"]
+++

User session management is essential for any web app and it has been reinvented so many times yet recently I had to write my [own session library](https://github.com/vividvilla/simplesessions) for Go. So here I am reasoning why I had to write one more.

## Backstory

At my firm we chose to use fasthttp for our apps since we had to scale for at least 100k concurrent users and unfortunately there isn’t any good session library for fasthttp. The one we were already using was [severely flawed](https://github.com/kataras/go-sessions/issues/18) and we ended up ditching it.

When we looked at other available options there were none and most of them supports only net/http which was not big of a surprise since fasthttp wasn’t used as heavily. So the idea was to write a session library for fasthttp but quickly realised that it will be better if its agnostic of networking library or even a framework.

## Secret sauce

The only way we could think of implementing is to offload cookie get and set to user as callbacks which pretty much the part depends on a networking library. For net/http in tuned out to be far simpler to implement callbacks and for fasthttp it was little verbose but still very straight forward. Here is an example for net/http callback implementation.

```go
func getCookie(name string, r interface{}) (*http.Cookie, error) {
 rd := r.(*http.Request)
 cookie, err := rd.Cookie(name)
 if err != nil {
  return nil, err
 }
 return cookie, nil
}

func setCookie(cookie *http.Cookie, w interface{}) error {
 // Get write interface registered using `Acquire` method in handlers.
 wr := w.(http.ResponseWriter)
 http.SetCookie(wr, cookie)
 return nil
}
```

## Design decisions

Apart from making it agnostic few other design decisions were made

- No implicit Encoding/decoding- Most of the session libs encode the data as a gob or JSON before storing it in backend and decoded after retrieving. While this is good if you need to store complex data structure in session but most of the time its just primitive data types like string, int or bool are stored. As said most of the libs does this implicitly and there is no way to disable and for apps like in our scale we couldn’t afford it. So its upto user to use whatever encoding/decoding needed and use sessions just to store and retrieve it.
- Heavy lifting is done by the store and session manager is just a scaffold. As said session libs implicitly encode and decode and use backend like redis to just store as a string. But it didn’t make sense to retrieve all the keys and values everytime any one session field is retrieved. For example hash maps can be used to hold single session and session fields can be mapped to hash map fields for efficiency. So in order to have flexibility and efficiency while storing the session data in backend bulk of the logic is offloaded to store and its upto store to implement it.

## Wrapping up

Since its my first Go library got a chance to learn a lot and had fun while developing it. I was able to achieve more than 98% test coverage which made me confident before shipping it and also got a chance to learn and implement go modules.

Most of the time was spent on coming up with the design and api than writing it which was very satisfying personally. Got few feedbacks from reddit and got to refactor few things which I feel made the lib better.

Would like to thank [Kailash](https://github.com/knadh) for helping me with tough design decisions and overall a great experience personally and can’t wait for people to use it in production apps.

> You can check the project on Github here — <https://github.com/vividvilla/simplesessions>
