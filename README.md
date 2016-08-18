# OAuth 2.0

>  An open protocol to allow secure authorization in a simple and standard method from web, mobile and desktop applications.

[![License][License-Image]][License-Url] 
[![ReportCard][ReportCard-Image]][ReportCard-Url] 
[![Build][Build-Status-Image]][Build-Status-Url] 
[![GoDoc][GoDoc-Image]][GoDoc-Url]
[![Release][Release-Image]][Release-Url] 

## Protocol Flow

```
     +--------+                               +---------------+
     |        |--(A)- Authorization Request ->|   Resource    |
     |        |                               |     Owner     |
     |        |<-(B)-- Authorization Grant ---|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(C)-- Authorization Grant -->| Authorization |
     | Client |                               |     Server    |
     |        |<-(D)----- Access Token -------|               |
     |        |                               +---------------+
     |        |
     |        |                               +---------------+
     |        |--(E)----- Access Token ------>|    Resource   |
     |        |                               |     Server    |
     |        |<-(F)--- Protected Resource ---|               |
     +--------+                               +---------------+
```

## Quick Start

### Download and install

``` bash
$ go get -u gopkg.in/oauth2.v3/...
```

### Create file `server.go`

``` go
package main

import (
	"fmt"
	"net/http"

	"gopkg.in/oauth2.v3/manage"
	"gopkg.in/oauth2.v3/server"
	"gopkg.in/oauth2.v3/store"
)

func main() {
	manager := manage.NewDefaultManager()
	// token memory store
	manager.MustTokenStorage(store.NewMemoryTokenStore())
	// client test store
	manager.MapClientStorage(store.NewTestClientStore())

	srv := server.NewDefaultServer(manager)
	srv.SetAllowGetAccessRequest(true)

	srv.SetInternalErrorHandler(func(err error) {
		fmt.Println("OAuth2 Error:",err.Error())
	})

	http.HandleFunc("/authorize", func(w http.ResponseWriter, r *http.Request) {
		err := srv.HandleAuthorizeRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusBadRequest)
		}
	})

	http.HandleFunc("/token", func(w http.ResponseWriter, r *http.Request) {
		err := srv.HandleTokenRequest(w, r)
		if err != nil {
			http.Error(w, err.Error(), http.StatusInternalServerError)
		}
	})

	http.ListenAndServe(":9096", nil)
}
```

### Build and run

``` bash
$ go build server.go
$ ./server
```

### Open in your web browser

```
http://localhost:9096/token?grant_type=clientcredentials&client_id=1&client_secret=11&scope=all
```

```
{
    "access_token": "ZGF4ARHJPT2Y_QAIOJVL-Q",
    "expires_in": 7200,
    "scope": "all",
    "token_type": "Bearer"
}
```

## Features

* Easy to use
* Based on the [RFC 6749](https://tools.ietf.org/html/rfc6749) implementation
* Token storage support TTL
* Support custom extension field
* Support custom scope
* Support custom expiration time of the access token

## Example

> A complete example of simulation authorization code model

Simulation examples of authorization code model, please check [example](/example)

## Storage Implements

* [BuntDB](https://github.com/tidwall/buntdb)(The default storage)
* [Redis](https://github.com/go-oauth2/redis)
* [MongoDB](https://github.com/go-oauth2/mongo)

## MIT License

```
Copyright (c) 2016 Lyric
```

[License-Url]: http://opensource.org/licenses/MIT
[License-Image]: https://img.shields.io/npm/l/express.svg
[Build-Status-Url]: https://travis-ci.org/go-oauth2/oauth2
[Build-Status-Image]: https://travis-ci.org/go-oauth2/oauth2.svg?branch=master
[Release-Url]: https://github.com/go-oauth2/oauth2/releases/tag/v3.4.8
[Release-image]: http://img.shields.io/badge/release-v3.4.8-1eb0fc.svg
[ReportCard-Url]: https://goreportcard.com/report/gopkg.in/oauth2.v3
[ReportCard-Image]: https://goreportcard.com/badge/gopkg.in/oauth2.v3
[GoDoc-Url]: https://godoc.org/gopkg.in/oauth2.v3
[GoDoc-Image]: https://godoc.org/gopkg.in/oauth2.v3?status.svg