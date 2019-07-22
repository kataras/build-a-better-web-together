Iris has the simplest known form for subdomains registration to a single application. Of course you can always use nginx or caddy for managment in production.

Subdomains are separated into two categories: **static** and **dynamic/wildcard**.

* Static : when you know the subdomain, i.e : `analytics.mydomain.com`
* Wildcard : when you don't know the subdomain but you know that it's before a particular subdomain or root domain, i.e : `user_created.mydomain.com`, `otheruser.mydomain.com` like the  `username.github.io`

We use the `Subdomain` and `WildcardSubdomain` methods of an `iris.Party` or `iris.Application` to register subdomains.

The `Subdomain` method returns a new `Party` which is responsible to register routes to
this specific "subdomain".

The only difference from a regular `Party` is that if called from a child party then the subdomain will be prepended to the path instead of appended.
So if `app.Subdomain("admin").Subdomain("panel")` then the result is: `"panel.admin."`.

```go
Subdomain(subdomain string, middleware ...Handler) Party
```

The `WildcardSubdomain` method returns a new `Party` which is responsible to register routes to
a dynamic, wildcard(ed) subdomain. A dynamic subdomain is a subdomain which
can handle any subdomain requests. Server will accept any subdomain
(if not static subdomain found) and it will search and execute the handlers of this `Party`.

```go
WildcardSubdomain(middleware ...Handler) Party
```

Example Code:

```go
// [app := iris.New...]
admin := app.Subdomain("admin")

// admin.mydomain.com
admin.Get("/", func(ctx iris.Context) {
    ctx.Writef("INDEX FROM admin.mydomain.com")
})

// admin.mydomain.com/hey
admin.Get("/hey", func(ctx iris.Context) {
    ctx.Writef("HEY FROM admin.mydomain.com/hey")
})

// [other routes here...]

app.Run(iris.Addr("mydomain.com:80"))
```

For local development you'll have to edit your hosts, for example in windows operating system open the `C:\Windows\System32\Drivers\etc\hosts` file and append:

```txt
127.0.0.1 mydomain.com
127.0.0.1 admin.mydomain.com
```

To prove that subdomains works like any other regular `Party` you can also register a subdomain using the alternative method below:

```go
adminSubdomain:= app.Party("admin.")
// or
adminAnalayticsSubdomain := app.Party("admin.analytics.")
// or for a dynamic one:
anySubdomain := app.Party("*.")
```

There is also an `iris.Application` method which allows to register a global redirection rule for subdomains as well.

The `SubdomainRedirect` sets (or adds if used more than one time) a router wrapper which
redirects(StatusMovedPermanently) a (sub)domain to another subdomain or to the root domain as fast as possible,
before the execution of the route's handler(s).

It receives two arguments, they are the from and to/target locations,
'from' can be a wildcard subdomain as well (app.WildcardSubdomain())
'to' is not allowed to be a wildcard for obvious reasons,
'from' can be the root domain(app) when the 'to' is not the root domain and visa-versa.

```go
SubdomainRedirect(from, to Party) Party
```

**Usage**

```go
www := app.Subdomain("www")
app.SubdomainRedirect(app, www)
```

The above will redirect all http(s)://mydomain.com/%anypath% to http(s)://www.mydomain.com/%anypath%.

The `Context` offers four main methods when working with subdomains that may be helpful for you.

```go
// Host returns the host part of the current url.
Host() string
// Subdomain returns the subdomain of this request, if any.
// Note that this is a fast method which does not cover all cases.
Subdomain() (subdomain string)
// IsWWW returns true if the current subdomain (if any) is www.
IsWWW() bool
// FullRqeuestURI returns the full URI,
// including the scheme, the host and the relative requested path/resource.
FullRequestURI() string
```

**Usage**

```go
func info(ctx iris.Context) {
    method := ctx.Method()
    subdomain := ctx.Subdomain()
    path := ctx.Path()

    ctx.Writef("\nInfo\n\n")
    ctx.Writef("Method: %s\nSubdomain: %s\nPath: %s", method, subdomain, path)
}
```
