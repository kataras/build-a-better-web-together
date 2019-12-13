# Sitemap

Iris provides extensive support for the Sitemap Protocol which automatically generates [sitemap index](https://www.sitemaps.org/protocol.html#index) "/sitemap.xml" too.

To enable sitemaps on your web application you should use the `iris.WithSitemap` Configurator. This function accepts the full scheme and domain of the exposed application.

```go
app := iris.New()
// [...]

app.Run(iris.Addr(":8080"), iris.WithSitemap("http://localhost:8080"))
```

The application will loop through registered _static_ routes and it will add each one of them to the sitemap file. By default only the `<loc>` XML element will be filled unless the route's fields `LastMod`, `ChangeFreq` or/and `Priority`[*](https://www.sitemaps.org/protocol.html) are set.

```go
app.Get("/home", handler).SetLastMod(time.Now()).SetChangeFreq("hourly").SetPriority(0.8)
```

> A static route is exposed on GET HTTP Method and its path does not contain a dynamic parameter. e.g. /home, /about but not /user/{id:uint64} and e.t.c.

See it in action, download and run the **example** from: https://github.com/kataras/iris/tree/master/_examples/sitemap

If the application **is localized** then `iris.WithSitemap` will add `xhtml:link` XML elements to the sitemap file for each translation language as recommended at: https://support.google.com/webmasters/answer/189077?hl=en. Read more about it at the [Localization](localization.md) section.
