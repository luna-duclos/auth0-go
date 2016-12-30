[![Build Status](https://travis-ci.org/yageek/auth0.svg?branch=develop)](https://travis-ci.org/yageek/auth0)
[![Coverage Status](https://coveralls.io/repos/github/yageek/auth0/badge.svg?branch=develop)](https://coveralls.io/github/yageek/auth0?branch=develop)
[![GoDoc](https://godoc.org/github.com/yageek/auth0?status.png)](https://godoc.org/github.com/yageek/auth0)
[![Report Cart](http://goreportcard.com/badge/yageek/auth0)](http://goreportcard.com/report/yageek/auth0)
[![MIT License](http://img.shields.io/badge/license-MIT-blue.svg?style=flat)](LICENSE)

# auth0

auth0 is a package helping to authenticate using the [Auth0](https://auth0.com) service.

## Installation 

```
go get github.com/yageek/auth0
```

## Client Credentials

```go
//Creates a configuration with the Auth0 information
secret, _ := base64.URLEncoding.DecodeString(os.Getenv("AUTH0_CLIENT_SECRET"))
secretProvider := auth0.NewKeyProvider([]byte("secret"))
audience := os.Getenv("AUTH0_CLIENT_ID")

configuration := auth0.NewConfiguration(secretProvider, audience, "https://mydomain.eu.auth0.com/", jose.RS256)
validator := auth0.NewValidator(configuration)



token, err := validator.ValidateRequest(r)

if err != nil {
    fmt.Println("Token is not valid:", token)
}
```

## API with JWK

```go
   
client := NewJWKClient(JWKClientOptions{URI: "https://mydomain.eu.auth0.com/.well-known/jwks.json"})
audience := os.Getenv("AUTH0_CLIENT_ID")
configuration := NewConfiguration(client, audience, "https://mydomain.eu.auth0.com/", jose.RS256)
validator := NewValidator(configuration)

token, err := validator.ValidateRequest(r)

if err != nil {
    fmt.Println("Token is not valid:", token)
}
```
## Example

### Gin

Using [Gin](https://github.com/gin-gonic/gin) and the [Auth0 Authorization Extension](https://auth0.com/docs/extensions/authorization-extension), you 
may want to implement the authentication auth like the following:

```go
var auth.AdminGroup string = "my_admin_group"

// Access Control Helper function.
func shouldAccess(wantedGroups []string, groups []interface{}) bool { 
 /* Fill depending on your needs */
}

// Wrapping a Gin endpoint with Auth0 Groups.
func Auth0Groups(wantedGroups ...string) gin.HandlerFunc {

	return gin.HandlerFunc(func(c *gin.Context) {
		jwt, err := validator.ValidateRequest(c.Request)

		if err != nil {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid token"})
			c.Abort()
			log.Println("InvalidToken:", jwt)
			return
		}
        
        claims := map[string]interface{}{}
        err = validator.Claims(c.Request, tok, &claims)
        
        if err != nil {
        			c.JSON(http.StatusUnauthorized, gin.H{"error": "invalid claims"})
        			c.Abort()
        			return
        		}
        		
		metadata, okMetadata := jwt.Claims().Get("app_metadata").(map[string]interface{})
		authorization, okAuthorization := metadata["authorization"].(map[string]interface{})
		groups, hasGroups := authorization["groups"].([]interface{})

		if !okMetadata || !okAuthorization || !hasGroups || !shouldAccess(wantedGroups, groups) {
			c.JSON(http.StatusUnauthorized, gin.H{"error": "need more privileges"})
			c.Abort()
			return
		}
		c.Next()
	})
}

// Use it
r.PUT("/news", auth.Auth0Groups(auth.AdminGroup), api.GetNews)
```
