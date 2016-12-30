# Secure

[This is a middleware](https://github.com/iris-contrib/middleware/tree/5.0.0/secure)

Secure is an HTTP middleware for Go that facilitates some quick security wins.

```go
import "github.com/iris-contrib/middleware/secure"

secure.New(secure.Options{}) // options here

```

Example

```go
package main

import (
    "gopkg.in/kataras/iris.v5"
    "github.com/iris-contrib/middleware/secure"
)

func main() {
    s := secure.New(secure.Options{
        // AllowedHosts is a list of fully qualified domain names
        // that are allowed. Default is empty list, 
        // which allows any and all host names.
        AllowedHosts:            []string{"ssl.example.com"},                                                                                                                         
        // If SSLRedirect is set to true, then only allow HTTPS requests.
        // Default is false.
        SSLRedirect:             true,     
        // If SSLTemporaryRedirect is true, 
        // then a 302 will be used while redirecting.
        // Default is false (301).
        SSLTemporaryRedirect:    false,    
        // SSLHost is the host name that is used to 
        // redirect HTTP requests to HTTPS.
        // Default is "", which indicates to use the same host.
        SSLHost:                 "ssl.example.com",
        // SSLProxyHeaders is set of header keys with associated values 
        // that would indicate a valid HTTPS request. 
        // Useful when using Nginx: `map[string]string{"X-Forwarded-Proto": "https"}`. 
        // Default is blank map.
        SSLProxyHeaders:         map[string]string{"X-Forwarded-Proto": "https"},
        // STSSeconds is the max-age of the Strict-Transport-Security header.
        // Default is 0, which would NOT include the header.
        STSIncludeSubdomains:    true,                                                                                                                                                
        // If STSIncludeSubdomains is set to true, 
        // the `includeSubdomains`
        // will be appended to the Strict-Transport-Security header. Default is false.
        STSSeconds:              315360000,                                                                                                                                           
        // If STSPreload is set to true, the `preload`
        // flag will be appended to the Strict-Transport-Security header.
        // Default is false.
        STSPreload:              true,    
        // STS header is only included when the connection is HTTPS. 
        // If you want to force it to always be added, set to true. 
        // `IsDevelopment` still overrides this. Default is false.
        ForceSTSHeader:          false,  
        // If FrameDeny is set to true, adds the X-Frame-Options header with
        // the value of `DENY`. Default is false.
        FrameDeny:               true,    
        // CustomFrameOptionsValue allows the X-Frame-Options header 
        // value to be set with a custom value. 
        // This overrides the FrameDeny option.
        CustomFrameOptionsValue: "SAMEORIGIN",
        // If ContentTypeNosniff is true, adds the X-Content-Type-Options
        // header with the value `nosniff`. Default is false.
        ContentTypeNosniff:      true,  
        // If BrowserXssFilter is true, adds the X-XSS-Protection header 
        // with the value `1;mode=block`. Default is false.
        BrowserXSSFilter:        true,
        // ContentSecurityPolicy allows the Content-Security-Policy
        // header value to be set with a custom value. Default is "".
        ContentSecurityPolicy:   "default-src 'self'",   
        // PublicKey implements HPKP to prevent 
        // MITM attacks with forged certificates. Default is "".
        PublicKey:               `pin-sha256="base64+primary=="; pin-sha256="base64+backup=="; max-age=5184000; includeSubdomains; report-uri="https://www.example.com/hpkp-report"`,
        // This will cause the AllowedHosts, SSLRedirect, 
        //..and STSSeconds/STSIncludeSubdomains options to be ignored during development. 
        // When deploying to production, be sure to set this to false.
        IsDevelopment: true,
    })

    iris.UseFunc(func(c *iris.Context) {
        err := s.Process(c)

        // If there was an error, do not continue.
        if err != nil {
            return
        }

        c.Next()
    })

    iris.Get("/home", func(c *iris.Context) {
        c.Write("Hello from /home")
    })

    iris.Listen(":8080")
}


```