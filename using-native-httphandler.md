# Using native http.Handler

> Not recommended.
> Note that using native http handler you cannot access url params.



```go

type nativehandler struct {}

func (_ nativehandler) ServeHTTP(res http.ResponseWriter, req *http.Request) {

}

func main() {
	iris.Handle("", "/path", iris.ToHandler(nativehandler{}))
	//"" means ANY(GET,POST,PUT,DELETE and so on)
}


```
