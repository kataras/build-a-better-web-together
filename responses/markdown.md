# Markdown

**Content-Type: "text/html"**

Iris can be used to parse and render markdown contents as HTML to the web client.

```go
Markdown(contents []byte, options ...Markdown) (int, error)
```

It accepts the raw `[]byte` contents and optionally `Markdown` options structure which contains a single `Sanitize bool` field.  If `Sanitize` is set to true then it takes a `[]byte` that contains a HTML fragment or document and applies the `UGCPolicy`. The UGCPolicy is a policy aimed at user generated content that is a result of `HTML WYSIWYG` tools and Markdown conversions. This is expected to be a fairly rich document where as much markup as possible should be retained. Markdown permits raw HTML so we are basically providing a policy to sanitise **HTML5 documents safely** but with the
least intrusion on the formatting expectations of the user.

```go
func handler(ctx iris.Context) {
    response := []byte(`# Hello Dynamic Markdown -- Iris`)
    ctx.Markdown(response)
}
```

**Result**

```html
<h1>Hello Dynamic Markdown &ndash; Iris</h1>
```
<!-- slide:break-80 -->
