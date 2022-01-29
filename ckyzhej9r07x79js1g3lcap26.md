## Auth0 JWT Middleware in Go - Gin Web Framework


In today's blog post, we will see how to validate the JWTs using Auth0 Golang JWT middleware using Gin Web Framework. Gin is a web framework written in Go (Golang). In deep-dive, we will see how to integrate the Auth0 Golang JWT middleware to verify JWTs generated using both HS256 and RS256 using secret and JWKs.

Auth0 Golang JWT middleware stable version v2.0.0 released on Jan 19, 2022

[Release v2.0.0 · auth0/go-jwt-middleware](https://github.com/auth0/go-jwt-middleware/releases/tag/v2.0.0)

## Gin Web Framework

Gin is a high-performance micro-framework that can build web applications and microservices. It makes it simple to build a request handling pipeline from modular, reusable pieces. It does this by allowing you to write middleware that can be plugged into one or more request handlers or groups of request handlers.

Let's create a simple API using Golang Gin Web Framework.

- Install Gin package
  ```bash
  go get -u github.com/gin-gonic/gin
  ```
- Import it in your code and run simple api

  ```go
  package main

  import (
  	"context"
  	"net/http"
  	"github.com/gin-gonic/gin"
  )

  type Product struct {
  	ID    int     `json:"id"`
  	Title string  `json:"title"`
  	Code  string  `json:"code"`
  	Price float32 `json:"price"`
  }

  func main() {
  	r := gin.Default()

  	r.GET("/products", func(c *gin.Context) {
  		products := []Product{
  			{ID: 1, Title: "Product 1", Code: "p1", Price: 100.0},
  			{ID: 2, Title: "Product 2", Code: "p2", Price: 200.0},
  			{ID: 3, Title: "Product 3", Code: "p3", Price: 300.0},
  		}
  		c.JSON(http.StatusOK, products)
  	})

  	// Listen and Server in 0.0.0.0:5000
  	r.Run(":5000")
  }
  ```

## Add JWT Middleware

Before configuring the Auth0 APIs, lets' integrate the Auth0 Golang JWT middleware.

[GitHub - auth0/go-jwt-middleware: A Middleware for Go Programming Language to check for JWTs on HTTP requests](https://github.com/auth0/go-jwt-middleware)

```go
go get github.com/auth0/go-jwt-middleware/v2
```

Auth0 Golang JWT middleware is the HTTP middleware handler. To use it in the Gin Web framework, we need a wrapper to wrap the common HTTP middleware to the Gin middleware handler. For that purpose, I'm using Gin Adapter from Gareth Watts. [https://github.com/gwatts/gin-adapter](https://github.com/gwatts/gin-adapter)

```go
package main

import (
	"context"
	"net/http"

	jwtmiddleware "github.com/auth0/go-jwt-middleware/v2"
	"github.com/auth0/go-jwt-middleware/v2/validator"
	"github.com/gin-gonic/gin"
	"github.com/gwatts/gin-adapter"
)

type Product struct {
	ID    int     `json:"id"`
	Title string  `json:"title"`
	Code  string  `json:"code"`
	Price float32 `json:"price"`
}

func main() {
	r := gin.Default()

	keyFunc := func(ctx context.Context) (interface{}, error) {
		return []byte("secret"), nil
	}

	jwtValidator, _ := validator.New(keyFunc, validator.HS256, "http://localhost:5000", []string{"api:read"})
	jwtMiddleware := jwtmiddleware.New(jwtValidator.ValidateToken)

	// Wrap the http handler with gin adapter
	r.Use(adapter.Wrap(jwtMiddleware.CheckJWT))

	r.GET("/products", func(c *gin.Context) {
		products := []Product{
			{ID: 1, Title: "Product 1", Code: "p1", Price: 100.0},
			{ID: 2, Title: "Product 2", Code: "p2", Price: 200.0},
			{ID: 3, Title: "Product 3", Code: "p3", Price: 300.0},
		}
		c.JSON(http.StatusOK, products)
	})

	// Listen and Server in 0.0.0.0:5000
	r.Run(":5000")
}
```

Let's test this API with JWT URLs.

```go
curl --location --request GET 'http://localhost:5000/products' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjUwMDAiLCJhdWQiOiJhcGk6cmVhZCJ9.hbgwKW_RILeXXDDUv5bVK3WgjtvqoK5IiuisgnFWefY'
```

You can generate a new JWT with issuer `[http://localhost:5000](http://localhost:5000)` and audience `api:read` with the HS256 algorithm with a shared secret `secret`

```bash
➜  ~  curl --location --request GET 'http://localhost:5000/products' \
--header 'Authorization: Bearer eyJhbGciOiJIUzI1NiIsInR5cCI6IkpXVCJ9.eyJzdWIiOiIxMjM0NTY3ODkwIiwibmFtZSI6IkpvaG4gRG9lIiwiaWF0IjoxNTE2MjM5MDIyLCJpc3MiOiJodHRwOi8vbG9jYWxob3N0OjUwMDAiLCJhdWQiOiJhcGk6cmVhZCJ9.hbgwKW_RILeXXDDUv5bVK3WgjtvqoK5IiuisgnFWefY' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   160  100   160    0     0   6011      0 --:--:-- --:--:-- --:--:-- 20000
[
  {
    "id": 1,
    "title": "Product 1",
    "code": "p1",
    "price": 100
  },
  {
    "id": 2,
    "title": "Product 2",
    "code": "p2",
    "price": 200
  },
  {
    "id": 3,
    "title": "Product 3",
    "code": "p3",
    "price": 300
  }
]
```

## Configure Auth0 APIs

Now it's time to add authorization to the API using Auth0. Let's configure Auth0 APIs with the signing algorithm RS256 and change the Auth0 Golang middleware code to support RS256.

![api.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643439306590/v_owRw7RM.png)

Add the permission `read:products` in the API

![scopes.png](https://cdn.hashnode.com/res/hashnode/image/upload/v1643439332320/COrvvVSvL-.png)

Change the Auth0 middleware to validate the RS256 signature JWT token.

```go
issuerURL, _ := url.Parse(os.Getenv("AUTH0_ISSUER_URL"))
audience := os.Getenv("AUTH0_AUDIENCE")

provider := jwks.NewCachingProvider(issuerURL, time.Duration(5*time.Minute))

jwtValidator, _ := validator.New(provider.KeyFunc,
	validator.RS256,
	issuerURL.String(),
	[]string{audience},
)

jwtMiddleware := jwtmiddleware.New(jwtValidator.ValidateToken)
r.Use(adapter.Wrap(jwtMiddleware.CheckJWT))
```

The complete code looks like below.

```go
package main

import (
	"log"
	"net/http"
	"net/url"
	"os"
	"time"

	adapter "github.com/gwatts/gin-adapter"

	jwtmiddleware "github.com/auth0/go-jwt-middleware/v2"
	"github.com/auth0/go-jwt-middleware/v2/jwks"
	"github.com/auth0/go-jwt-middleware/v2/validator"

	"github.com/gin-gonic/gin"
	"github.com/joho/godotenv"
)

type Product struct {
	ID    int     `json:"id"`
	Title string  `json:"title"`
	Code  string  `json:"code"`
	Price float32 `json:"price"`
}

func main() {
	err := godotenv.Load()
	if err != nil {
		log.Fatal("Error loading .env file")
	}

	r := gin.Default()

	issuerURL, _ := url.Parse(os.Getenv("AUTH0_ISSUER_URL"))
	audience := os.Getenv("AUTH0_AUDIENCE")

	provider := jwks.NewCachingProvider(issuerURL, time.Duration(5*time.Minute))

	jwtValidator, _ := validator.New(provider.KeyFunc,
		validator.RS256,
		issuerURL.String(),
		[]string{audience},
	)

	jwtMiddleware := jwtmiddleware.New(jwtValidator.ValidateToken)
	r.Use(adapter.Wrap(jwtMiddleware.CheckJWT))

	r.GET("/products", func(c *gin.Context) {
		products := []Product{
			{ID: 1, Title: "Product 1", Code: "p1", Price: 100.0},
			{ID: 2, Title: "Product 2", Code: "p2", Price: 200.0},
			{ID: 3, Title: "Product 3", Code: "p3", Price: 300.0},
		}
		c.JSON(http.StatusOK, products)
	})

	// Listen and Server in 0.0.0.0:5000
	r.Run(":5000")
}
```

## Verify Auth0 JWT

Create a test JWT from the API Test page. Call the API with the test token.

```bash
➜  auth0-go-gin-middleware  curl --location --request GET 'http://localhost:5000/products' \
--header 'authorization: Bearer eyJhbGciOiJSUzI1NiIsInR5cCI6IkpXVCIsImtpZCI6InVUT0ktNGhrSDBWNU9YUGxKV0xpXyJ9.eyJpc3MiOiJodHRwczovL3NpdmEtZGVtby1hcHAudXMuYXV0aDAuY29tLyIsInN1YiI6Inl2N1NDekVueUxTWGdMZ1d3b2pJODZvNk5ZMzh0cmNtQGNsaWVudHMiLCJhdWQiOiJodHRwczovL3Byb2R1Y3RzLWFwaS8iLCJpYXQiOjE2NDM0MzM4NTYsImV4cCI6MTY0MzUyMDI1NiwiYXpwIjoieXY3U0N6RW55TFNYZ0xnV3dvakk4Nm82TlkzOHRyY20iLCJndHkiOiJjbGllbnQtY3JlZGVudGlhbHMifQ.yH13H5XxLEABu3o8s2HZUs8Q9PXHeLGUcELQdlrYoClKP3k3B_WdUpaH2_c-UpA1ZjB_Is71-hSt3iqQN_OaMV_fFqpnt0qJQNXsoWSHBE5CzDDAclRlFf5XaWVbcA072rzUAtrJuPzHYf8kdR91243lJFA_13V5vlQuWxqFxas4FonVR5OLGcXYHqLfBI76DPfFaOBwOzefSYSI_jxKrrQtnux4Ktkqrgo7tpFckY6UfD6fXPeRvk4xLSb_SteAwcpCQrWJVyt7gTWv4_KkSNEyZduEbMTrnkU_jdQjHuKOLaTBc4t7t3MK_2_tmrda5xn0QXf-1y6P2M4zQu2mWA' | jq
  % Total    % Received % Xferd  Average Speed   Time    Time     Time  Current
                                 Dload  Upload   Total   Spent    Left  Speed
100   160  100   160    0     0   8045      0 --:--:-- --:--:-- --:--:-- 40000
[
  {
    "id": 1,
    "title": "Product 1",
    "code": "p1",
    "price": 100
  },
  {
    "id": 2,
    "title": "Product 2",
    "code": "p2",
    "price": 200
  },
  {
    "id": 3,
    "title": "Product 3",
    "code": "p3",
    "price": 300
  }
]
```

## Conclusion

In this blog post, we walked through how to use Auth0 Golang middleware v2.0.0 to verify the JWT on both HS256 and RS256 signatures. You can extend this demo project to verify the custom claims in JWT and more. We will see in more detail in upcoming blog posts.
