# All Context Methods

Here is a full list of methods that the `iris.Context` provides.

```go
type (
    // BodyDecoder is an interface which any struct can implement in order to customize the decode action
    // from ReadJSON and ReadXML
    //
    // Trivial example of this could be:
    // type User struct { Username string }
    //
    // func (u *User) Decode(data []byte) error {
    //	  return json.Unmarshal(data, u)
    // }
    //
    // the 'context.ReadJSON/ReadXML(&User{})' will call the User's
    // Decode option to decode the request body
    //
    // Note: This is totally optionally, the default decoders
    // for ReadJSON is the encoding/json and for ReadXML is the encoding/xml.
    BodyDecoder interface {
        Decode(data []byte) error
    }

    // Unmarshaler is the interface implemented by types that can unmarshal any raw data.
    // TIP INFO: Any pointer to a value which implements the BodyDecoder can be override the unmarshaler.
    Unmarshaler interface {
        Unmarshal(data []byte, outPtr interface{}) error
    }

    // UnmarshalerFunc a shortcut for the Unmarshaler interface
    //
    // See 'Unmarshaler' and 'BodyDecoder' for more.
    UnmarshalerFunc func(data []byte, outPtr interface{}) error
)

// Unmarshal parses the X-encoded data and stores the result in the value pointed to by v.
// Unmarshal uses the inverse of the encodings that Marshal uses, allocating maps,
// slices, and pointers as necessary.
func (u UnmarshalerFunc) Unmarshal(data []byte, v interface{}) error {
	return u(data, v)
}

// Context is the midle-man server's "object" for the clients.
//
// A New context is being acquired from a sync.Pool on each connection.
// The Context is the most important thing on the iris's http flow.
//
// Developers send responses to the client's request through a Context.
// Developers get request information from the client's request a Context.
type Context interface {
    // BeginRequest is executing once for each request
    // it should prepare the (new or acquired from pool) context's fields for the new request.
    //
    // To follow the iris' flow, developer should:
    // 1. reset handlers to nil
    // 2. reset values to empty
    // 3. reset sessions to nil
    // 4. reset response writer to the http.ResponseWriter
    // 5. reset request to the *http.Request
    // and any other optional steps, depends on dev's application type.
    BeginRequest(http.ResponseWriter, *http.Request)
    // EndRequest is executing once after a response to the request was sent and this context is useless or released.
    //
    // To follow the iris' flow, developer should:
    // 1. flush the response writer's result
    // 2. release the response writer
    // and any other optional steps, depends on dev's application type.
    EndRequest()

    // ResponseWriter returns an http.ResponseWriter compatible response writer, as expected.
    ResponseWriter() ResponseWriter
    // ResetResponseWriter should change or upgrade the Context's ResponseWriter.
    ResetResponseWriter(ResponseWriter)

    // Request returns the original *http.Request, as expected.
    Request() *http.Request
    // ResetRequest sets the Context's Request,
    // It is useful to store the new request created by a std *http.Request#WithContext() into Iris' Context.
    // Use `ResetRequest` when for some reason you want to make a full
    // override of the *http.Request.
    // Note that: when you just want to change one of each fields you can use the Request() which returns a pointer to Request,
    // so the changes will have affect without a full override.
    // Usage: you use a native http handler which uses the standard "context" package
    // to get values instead of the Iris' Context#Values():
    // r := ctx.Request()
    // stdCtx := context.WithValue(r.Context(), key, val)
    // ctx.ResetRequest(r.WithContext(stdCtx)).
    ResetRequest(r *http.Request)

    // SetCurrentRouteName sets the route's name internally,
    // in order to be able to find the correct current "read-only" Route when
    // end-developer calls the `GetCurrentRoute()` function.
    // It's being initialized by the Router, if you change that name
    // manually nothing really happens except that you'll get other
    // route via `GetCurrentRoute()`.
    // Instead, to execute a different path
    // from this context you should use the `Exec` function
    // or change the handlers via `SetHandlers/AddHandler` functions.
    SetCurrentRouteName(currentRouteName string)
    // GetCurrentRoute returns the current registered "read-only" route that
    // was being registered to this request's path.
    GetCurrentRoute() RouteReadOnly

    // Do calls the SetHandlers(handlers)
    // and executes the first handler,
    // handlers should not be empty.
    //
    // It's used by the router, developers may use that
    // to replace and execute handlers immediately.
    Do(Handlers)

    // AddHandler can add handler(s)
    // to the current request in serve-time,
    // these handlers are not persistenced to the router.
    //
    // Router is calling this function to add the route's handler.
    // If AddHandler called then the handlers will be inserted
    // to the end of the already-defined route's handler.
    //
    AddHandler(...Handler)
    // SetHandlers replaces all handlers with the new.
    SetHandlers(Handlers)
    // Handlers keeps tracking of the current handlers.
    Handlers() Handlers

    // HandlerIndex sets the current index of the
    // current context's handlers chain.
    // If -1 passed then it just returns the
    // current handler index without change the current index.
    //
    // Look Handlers(), Next() and StopExecution() too.
    HandlerIndex(n int) (currentIndex int)
    // Proceed is an alternative way to check if a particular handler
    // has been executed and called the `ctx.Next` function inside it.
    // This is useful only when you run a handler inside
    // another handler. It justs checks for before index and the after index.
    //
    // A usecase example is when you want to execute a middleware
    // inside controller's `BeginRequest` that calls the `ctx.Next` inside it.
    // The Controller looks the whole flow (BeginRequest, method handler, EndRequest)
    // as one handler, so `ctx.Next` will not be reflected to the method handler
    // if called from the `BeginRequest`.
    //
    // Although `BeginRequest` should NOT be used to call other handlers,
    // the `BeginRequest` has been introduced to be able to set
    // common data to all method handlers before their execution.
    // Controllers can accept middleware(s) from the MVC's Application's Router as normally.
    //
    // That said let's see an example of `ctx.Proceed`:
    //
    // var authMiddleware = basicauth.New(basicauth.Config{
    // 	Users: map[string]string{
    // 		"admin": "password",
    // 	},
    // })
    //
    // func (c *UsersController) BeginRequest(ctx iris.Context) {
    // 	if !ctx.Proceed(authMiddleware) {
    // 		ctx.StopExecution()
    // 	}
    // }
    // This Get() will be executed in the same handler as `BeginRequest`,
    // internally controller checks for `ctx.StopExecution`.
    // So it will not be fired if BeginRequest called the `StopExecution`.
    // func(c *UsersController) Get() []models.User {
    //	  return c.Service.GetAll()
    //}
    // Alternative way is `!ctx.IsStopped()` if middleware make use of the `ctx.StopExecution()` on failure.
    Proceed(Handler) bool
    // HandlerName returns the current handler's name, helpful for debugging.
    HandlerName() string
    // HandlerFileLine returns the current running handler's function source file and line information.
    // Useful mostly when debugging.
    HandlerFileLine() (file string, line int)
    // RouteName returns the route name that this handler is running on.
    // Note that it will return empty on not found handlers.
    RouteName() string
    // Next calls all the next handler from the handlers chain,
    // it should be used inside a middleware.
    //
    // Note: Custom context should override this method in order to be able to pass its own context.Context implementation.
    Next()
    // NextOr checks if chain has a next handler, if so then it executes it
    // otherwise it sets a new chain assigned to this Context based on the given handler(s)
    // and executes its first handler.
    //
    // Returns true if next handler exists and executed, otherwise false.
    //
    // Note that if no next handler found and handlers are missing then
    // it sends a Status Not Found (404) to the client and it stops the execution.
    NextOr(handlers ...Handler) bool
    // NextOrNotFound checks if chain has a next handler, if so then it executes it
    // otherwise it sends a Status Not Found (404) to the client and stops the execution.
    //
    // Returns true if next handler exists and executed, otherwise false.
    NextOrNotFound() bool
    // NextHandler returns (it doesn't execute) the next handler from the handlers chain.
    //
    // Use .Skip() to skip this handler if needed to execute the next of this returning handler.
    NextHandler() Handler
    // Skip skips/ignores the next handler from the handlers chain,
    // it should be used inside a middleware.
    Skip()
    // StopExecution if called then the following .Next calls are ignored,
    // as a result the next handlers in the chain will not be fire.
    StopExecution()
    // IsStopped checks and returns true if the current position of the Context is 255,
    // means that the StopExecution() was called.
    IsStopped() bool
    // OnConnectionClose registers the "cb" function which will fire (on its own goroutine, no need to be registered goroutine by the end-dev)
    // when the underlying connection has gone away.
    //
    // This mechanism can be used to cancel long operations on the server
    // if the client has disconnected before the response is ready.
    //
    // It depends on the `http#CloseNotify`.
    // CloseNotify may wait to notify until Request.Body has been
    // fully read.
    //
    // After the main Handler has returned, there is no guarantee
    // that the channel receives a value.
    //
    // Finally, it reports whether the protocol supports pipelines (HTTP/1.1 with pipelines disabled is not supported).
    // The "cb" will not fire for sure if the output value is false.
    //
    // Note that you can register only one callback for the entire request handler chain/per route.
    //
    // Look the `ResponseWriter#CloseNotifier` for more.
    OnConnectionClose(fnGoroutine func()) bool
    // OnClose registers the callback function "cb" to the underline connection closing event using the `Context#OnConnectionClose`
    // and also in the end of the request handler using the `ResponseWriter#SetBeforeFlush`.
    // Note that you can register only one callback for the entire request handler chain/per route.
    //
    // Look the `Context#OnConnectionClose` and `ResponseWriter#SetBeforeFlush` for more.
    OnClose(cb func())

    //  +------------------------------------------------------------+
    //  | Current "user/request" storage                             |
    //  | and share information between the handlers - Values().     |
    //  | Save and get named path parameters - Params()              |
    //  +------------------------------------------------------------+

    // Params returns the current url's named parameters key-value storage.
    // Named path parameters are being saved here.
    // This storage, as the whole Context, is per-request lifetime.
    Params() *RequestParams

    // Values returns the current "user" storage.
    // Named path parameters and any optional data can be saved here.
    // This storage, as the whole Context, is per-request lifetime.
    //
    // You can use this function to Set and Get local values
    // that can be used to share information between handlers and middleware.
    Values() *memstore.Store
    // Translate is the i18n (localization) middleware's function,
    // it calls the Values().Get(ctx.Application().ConfigurationReadOnly().GetTranslateFunctionContextKey())
    // to execute the translate function and return the localized text value.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/miscellaneous/i18n
    Translate(format string, args ...interface{}) string

    //  +------------------------------------------------------------+
    //  | Path, Host, Subdomain, IP, Headers etc...                  |
    //  +------------------------------------------------------------+

    // Method returns the request.Method, the client's http method to the server.
    Method() string
    // Path returns the full request path,
    // escaped if EnablePathEscape config field is true.
    Path() string
    // RequestPath returns the full request path,
    // based on the 'escape'.
    RequestPath(escape bool) string
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
    // RemoteAddr tries to parse and return the real client's request IP.
    //
    // Based on allowed headers names that can be modified from Configuration.RemoteAddrHeaders.
    //
    // If parse based on these headers fail then it will return the Request's `RemoteAddr` field
    // which is filled by the server before the HTTP handler.
    //
    // Look `Configuration.RemoteAddrHeaders`,
    //      `Configuration.WithRemoteAddrHeader(...)`,
    //      `Configuration.WithoutRemoteAddrHeader(...)` for more.
    RemoteAddr() string
    // GetHeader returns the request header's value based on its name.
    GetHeader(name string) string
    // IsAjax returns true if this request is an 'ajax request'( XMLHttpRequest)
    //
    // There is no a 100% way of knowing that a request was made via Ajax.
    // You should never trust data coming from the client, they can be easily overcome by spoofing.
    //
    // Note that "X-Requested-With" Header can be modified by any client(because of "X-"),
    // so don't rely on IsAjax for really serious stuff,
    // try to find another way of detecting the type(i.e, content type),
    // there are many blogs that describe these problems and provide different kind of solutions,
    // it's always depending on the application you're building,
    // this is the reason why this `IsAjax`` is simple enough for general purpose use.
    //
    // Read more at: https://developer.mozilla.org/en-US/docs/AJAX
    // and https://xhr.spec.whatwg.org/
    IsAjax() bool
    // IsMobile checks if client is using a mobile device(phone or tablet) to communicate with this server.
    // If the return value is true that means that the http client using a mobile
    // device to communicate with the server, otherwise false.
    //
    // Keep note that this checks the "User-Agent" request header.
    IsMobile() bool
    // GetReferrer extracts and returns the information from the "Referer" header as specified
    // in https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy
    // or by the URL query parameter "referer".
    GetReferrer() Referrer
    //  +------------------------------------------------------------+
    //  | Headers helpers                                            |
    //  +------------------------------------------------------------+

    // Header adds a header to the response writer.
    Header(name string, value string)

    // ContentType sets the response writer's header key "Content-Type" to the 'cType'.
    ContentType(cType string)
    // GetContentType returns the response writer's header value of "Content-Type"
    // which may, set before with the 'ContentType'.
    GetContentType() string
    // GetContentType returns the request's header value of "Content-Type".
    GetContentTypeRequested() string

    // GetContentLength returns the request's header value of "Content-Length".
    // Returns 0 if header was unable to be found or its value was not a valid number.
    GetContentLength() int64

    // StatusCode sets the status code header to the response.
    // Look .`GetStatusCode` too.
    StatusCode(statusCode int)
    // GetStatusCode returns the current status code of the response.
    // Look `StatusCode` too.
    GetStatusCode() int

    // Redirect sends a redirect response to the client
    // to a specific url or relative path.
    // accepts 2 parameters string and an optional int
    // first parameter is the url to redirect
    // second parameter is the http status should send,
    // default is 302 (StatusFound),
    // you can set it to 301 (Permant redirect)
    // or 303 (StatusSeeOther) if POST method,
    // or StatusTemporaryRedirect(307) if that's nessecery.
    Redirect(urlToRedirect string, statusHeader ...int)

    //  +------------------------------------------------------------+
    //  | Various Request and Post Data                              |
    //  +------------------------------------------------------------+

    // URLParam returns true if the url parameter exists, otherwise false.
    URLParamExists(name string) bool
    // URLParamDefault returns the get parameter from a request,
    // if not found then "def" is returned.
    URLParamDefault(name string, def string) string
    // URLParam returns the get parameter from a request, if any.
    URLParam(name string) string
    // URLParamTrim returns the url query parameter with trailing white spaces removed from a request.
    URLParamTrim(name string) string
    // URLParamTrim returns the escaped url query parameter from a request.
    URLParamEscape(name string) string
    // URLParamInt returns the url query parameter as int value from a request,
    // returns -1 and an error if parse failed.
    URLParamInt(name string) (int, error)
    // URLParamIntDefault returns the url query parameter as int value from a request,
    // if not found or parse failed then "def" is returned.
    URLParamIntDefault(name string, def int) int
    // URLParamInt32Default returns the url query parameter as int32 value from a request,
    // if not found or parse failed then "def" is returned.
    URLParamInt32Default(name string, def int32) int32
    // URLParamInt64 returns the url query parameter as int64 value from a request,
    // returns -1 and an error if parse failed.
    URLParamInt64(name string) (int64, error)
    // URLParamInt64Default returns the url query parameter as int64 value from a request,
    // if not found or parse failed then "def" is returned.
    URLParamInt64Default(name string, def int64) int64
    // URLParamFloat64 returns the url query parameter as float64 value from a request,
    // returns -1 and an error if parse failed.
    URLParamFloat64(name string) (float64, error)
    // URLParamFloat64Default returns the url query parameter as float64 value from a request,
    // if not found or parse failed then "def" is returned.
    URLParamFloat64Default(name string, def float64) float64
    // URLParamBool returns the url query parameter as boolean value from a request,
    // returns an error if parse failed or not found.
    URLParamBool(name string) (bool, error)
    // URLParams returns a map of GET query parameters separated by comma if more than one
    // it returns an empty map if nothing found.
    URLParams() map[string]string

    // FormValueDefault returns a single parsed form value by its "name",
    // including both the URL field's query parameters and the POST or PUT form data.
    //
    // Returns the "def" if not found.
    FormValueDefault(name string, def string) string
    // FormValue returns a single parsed form value by its "name",
    // including both the URL field's query parameters and the POST or PUT form data.
    FormValue(name string) string
    // FormValues returns the parsed form data, including both the URL
    // field's query parameters and the POST or PUT form data.
    //
    // The default form's memory maximum size is 32MB, it can be changed by the
    // `iris#WithPostMaxMemory` configurator at main configuration passed on `app.Run`'s second argument.
    //
    // NOTE: A check for nil is necessary.
    FormValues() map[string][]string

    // PostValueDefault returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name".
    //
    // If not found then "def" is returned instead.
    PostValueDefault(name string, def string) string
    // PostValue returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name"
    PostValue(name string) string
    // PostValueTrim returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name",  without trailing spaces.
    PostValueTrim(name string) string
    // PostValueInt returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as int.
    //
    // If not found returns -1 and a non-nil error.
    PostValueInt(name string) (int, error)
    // PostValueIntDefault returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as int.
    //
    // If not found returns or parse errors the "def".
    PostValueIntDefault(name string, def int) int
    // PostValueInt64 returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as float64.
    //
    // If not found returns -1 and a no-nil error.
    PostValueInt64(name string) (int64, error)
    // PostValueInt64Default returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as int64.
    //
    // If not found or parse errors returns the "def".
    PostValueInt64Default(name string, def int64) int64
    // PostValueInt64Default returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as float64.
    //
    // If not found returns -1 and a non-nil error.
    PostValueFloat64(name string) (float64, error)
    // PostValueInt64Default returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as float64.
    //
    // If not found or parse errors returns the "def".
    PostValueFloat64Default(name string, def float64) float64
    // PostValueInt64Default returns the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name", as bool.
    //
    // If not found or value is false, then it returns false, otherwise true.
    PostValueBool(name string) (bool, error)
    // PostValues returns all the parsed form data from POST, PATCH,
    // or PUT body parameters based on a "name" as a string slice.
    //
    // The default form's memory maximum size is 32MB, it can be changed by the
    // `iris#WithPostMaxMemory` configurator at main configuration passed on `app.Run`'s second argument.
    PostValues(name string) []string
    // FormFile returns the first uploaded file that received from the client.
    //
    // The default form's memory maximum size is 32MB, it can be changed by the
    //  `iris#WithPostMaxMemory` configurator at main configuration passed on `app.Run`'s second argument.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/http_request/upload-file
    FormFile(key string) (multipart.File, *multipart.FileHeader, error)
    // UploadFormFiles uploads any received file(s) from the client
    // to the system physical location "destDirectory".
    //
    // The second optional argument "before" gives caller the chance to
    // modify the *miltipart.FileHeader before saving to the disk,
    // it can be used to change a file's name based on the current request,
    // all FileHeader's options can be changed. You can ignore it if
    // you don't need to use this capability before saving a file to the disk.
    //
    // Note that it doesn't check if request body streamed.
    //
    // Returns the copied length as int64 and
    // a not nil error if at least one new file
    // can't be created due to the operating system's permissions or
    // http.ErrMissingFile if no file received.
    //
    // If you want to receive & accept files and manage them manually you can use the `context#FormFile`
    // instead and create a copy function that suits your needs, the below is for generic usage.
    //
    // The default form's memory maximum size is 32MB, it can be changed by the
    //  `iris#WithPostMaxMemory` configurator at main configuration passed on `app.Run`'s second argument.
    //
    // See `FormFile` to a more controlled to receive a file.
    //
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/http_request/upload-files
    UploadFormFiles(destDirectory string, before ...func(Context, *multipart.FileHeader)) (n int64, err error)

    //  +------------------------------------------------------------+
    //  | Custom HTTP Errors                                         |
    //  +------------------------------------------------------------+

    // NotFound emits an error 404 to the client, using the specific custom error error handler.
    // Note that you may need to call ctx.StopExecution() if you don't want the next handlers
    // to be executed. Next handlers are being executed on iris because you can alt the
    // error code and change it to a more specific one, i.e
    // users := app.Party("/users")
    // users.Done(func(ctx context.Context){ if ctx.StatusCode() == 400 { /*  custom error code for /users */ }})
    NotFound()

    //  +------------------------------------------------------------+
    //  | Body Readers                                               |
    //  +------------------------------------------------------------+

    // SetMaxRequestBodySize sets a limit to the request body size
    // should be called before reading the request body from the client.
    SetMaxRequestBodySize(limitOverBytes int64)

    // GetBody reads and returns the request body.
    // The default behavior for the http request reader is to consume the data readen
    // but you can change that behavior by passing the `WithoutBodyConsumptionOnUnmarshal` iris option.
    //
    // However, whenever you can use the `ctx.Request().Body` instead.
    GetBody() ([]byte, error)
    // UnmarshalBody reads the request's body and binds it to a value or pointer of any type.
    // Examples of usage: context.ReadJSON, context.ReadXML.
    //
    // Example: https://github.com/kataras/iris/blob/master/_examples/http_request/read-custom-via-unmarshaler/main.go
    //
    // UnmarshalBody does not check about gzipped data.
    // Do not rely on compressed data incoming to your server. The main reason is: https://en.wikipedia.org/wiki/Zip_bomb
    // However you are still free to read the `ctx.Request().Body io.Reader` manually.
    UnmarshalBody(outPtr interface{}, unmarshaler Unmarshaler) error
    // ReadJSON reads JSON from request's body and binds it to a pointer of a value of any json-valid type.
    //
    // Example: https://github.com/kataras/iris/blob/master/_examples/http_request/read-json/main.go
    ReadJSON(jsonObjectPtr interface{}) error
    // ReadXML reads XML from request's body and binds it to a pointer of a value of any xml-valid type.
    //
    // Example: https://github.com/kataras/iris/blob/master/_examples/http_request/read-xml/main.go
    ReadXML(xmlObjectPtr interface{}) error
    // ReadForm binds the formObject  with the form data
    // it supports any kind of type, including custom structs.
    // It will return nothing if request data are empty.
    // The struct field tag is "form".
    //
    // Example: https://github.com/kataras/iris/blob/master/_examples/http_request/read-form/main.go
    ReadForm(formObject interface{}) error
    // ReadQuery binds the "ptr" with the url query string. The struct field tag is "url".
    //
    // Example: https://github.com/kataras/iris/blob/master/_examples/http_request/read-query/main.go
    ReadQuery(ptr interface{}) error
    //  +------------------------------------------------------------+
    //  | Body (raw) Writers                                         |
    //  +------------------------------------------------------------+

    // Write writes the data to the connection as part of an HTTP reply.
    //
    // If WriteHeader has not yet been called, Write calls
    // WriteHeader(http.StatusOK) before writing the data. If the Header
    // does not contain a Content-Type line, Write adds a Content-Type set
    // to the result of passing the initial 512 bytes of written data to
    // DetectContentType.
    //
    // Depending on the HTTP protocol version and the client, calling
    // Write or WriteHeader may prevent future reads on the
    // Request.Body. For HTTP/1.x requests, handlers should read any
    // needed request body data before writing the response. Once the
    // headers have been flushed (due to either an explicit Flusher.Flush
    // call or writing enough data to trigger a flush), the request body
    // may be unavailable. For HTTP/2 requests, the Go HTTP server permits
    // handlers to continue to read the request body while concurrently
    // writing the response. However, such behavior may not be supported
    // by all HTTP/2 clients. Handlers should read before writing if
    // possible to maximize compatibility.
    Write(body []byte) (int, error)
    // Writef formats according to a format specifier and writes to the response.
    //
    // Returns the number of bytes written and any write error encountered.
    Writef(format string, args ...interface{}) (int, error)
    // WriteString writes a simple string to the response.
    //
    // Returns the number of bytes written and any write error encountered.
    WriteString(body string) (int, error)

    // SetLastModified sets the "Last-Modified" based on the "modtime" input.
    // If "modtime" is zero then it does nothing.
    //
    // It's mostly internally on core/router and context packages.
    //
    // Note that modtime.UTC() is being used instead of just modtime, so
    // you don't have to know the internals in order to make that works.
    SetLastModified(modtime time.Time)
    // CheckIfModifiedSince checks if the response is modified since the "modtime".
    // Note that it has nothing to do with server-side caching.
    // It does those checks by checking if the "If-Modified-Since" request header
    // sent by client or a previous server response header
    // (e.g with WriteWithExpiration or HandleDir or Favicon etc.)
    // is a valid one and it's before the "modtime".
    //
    // A check for !modtime && err == nil is necessary to make sure that
    // it's not modified since, because it may return false but without even
    // had the chance to check the client-side (request) header due to some errors,
    // like the HTTP Method is not "GET" or "HEAD" or if the "modtime" is zero
    // or if parsing time from the header failed.
    //
    // It's mostly used internally, e.g. `context#WriteWithExpiration`.
    //
    // Note that modtime.UTC() is being used instead of just modtime, so
    // you don't have to know the internals in order to make that works.
    CheckIfModifiedSince(modtime time.Time) (bool, error)
    // WriteNotModified sends a 304 "Not Modified" status code to the client,
    // it makes sure that the content type, the content length headers
    // and any "ETag" are removed before the response sent.
    //
    // It's mostly used internally on core/router/fs.go and context methods.
    WriteNotModified()
    // WriteWithExpiration works like `Write` but it will check if a resource is modified,
    // based on the "modtime" input argument,
    // otherwise sends a 304 status code in order to let the client-side render the cached content.
    WriteWithExpiration(body []byte, modtime time.Time) (int, error)
    // StreamWriter registers the given stream writer for populating
    // response body.
    //
    // Access to context's and/or its' members is forbidden from writer.
    //
    // This function may be used in the following cases:
    //
    //     * if response body is too big (more than iris.LimitRequestBodySize(if set)).
    //     * if response body is streamed from slow external sources.
    //     * if response body must be streamed to the client in chunks.
    //     (aka `http server push`).
    //
    // receives a function which receives the response writer
    // and returns false when it should stop writing, otherwise true in order to continue
    StreamWriter(writer func(w io.Writer) bool)

    //  +------------------------------------------------------------+
    //  | Body Writers with compression                              |
    //  +------------------------------------------------------------+
    // ClientSupportsGzip retruns true if the client supports gzip compression.
    ClientSupportsGzip() bool
    // WriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
    // returns the number of bytes written and an error ( if the client doesn' supports gzip compression)
    // You may re-use this function in the same handler
    // to write more data many times without any troubles.
    WriteGzip(b []byte) (int, error)
    // TryWriteGzip accepts bytes, which are compressed to gzip format and sent to the client.
    // If client does not supprots gzip then the contents are written as they are, uncompressed.
    TryWriteGzip(b []byte) (int, error)
    // GzipResponseWriter converts the current response writer into a response writer
    // which when its .Write called it compress the data to gzip and writes them to the client.
    //
    // Can be also disabled with its .Disable and .ResetBody to rollback to the usual response writer.
    GzipResponseWriter() *GzipResponseWriter
    // Gzip enables or disables (if enabled before) the gzip response writer,if the client
    // supports gzip compression, so the following response data will
    // be sent as compressed gzip data to the client.
    Gzip(enable bool)

    //  +------------------------------------------------------------+
    //  | Rich Body Content Writers/Renderers                        |
    //  +------------------------------------------------------------+

    // ViewLayout sets the "layout" option if and when .View
    // is being called afterwards, in the same request.
    // Useful when need to set or/and change a layout based on the previous handlers in the chain.
    //
    // Note that the 'layoutTmplFile' argument can be set to iris.NoLayout || view.NoLayout
    // to disable the layout for a specific view render action,
    // it disables the engine's configuration's layout property.
    //
    // Look .ViewData and .View too.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/view/context-view-data/
    ViewLayout(layoutTmplFile string)
    // ViewData saves one or more key-value pair in order to be passed if and when .View
    // is being called afterwards, in the same request.
    // Useful when need to set or/and change template data from previous hanadlers in the chain.
    //
    // If .View's "binding" argument is not nil and it's not a type of map
    // then these data are being ignored, binding has the priority, so the main route's handler can still decide.
    // If binding is a map or context.Map then these data are being added to the view data
    // and passed to the template.
    //
    // After .View, the data are not destroyed, in order to be re-used if needed (again, in the same request as everything else),
    // to clear the view data, developers can call:
    // ctx.Set(ctx.Application().ConfigurationReadOnly().GetViewDataContextKey(), nil)
    //
    // If 'key' is empty then the value is added as it's (struct or map) and developer is unable to add other value.
    //
    // Look .ViewLayout and .View too.
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/view/context-view-data/
    ViewData(key string, value interface{})
    // GetViewData returns the values registered by `context#ViewData`.
    // The return value is `map[string]interface{}`, this means that
    // if a custom struct registered to ViewData then this function
    // will try to parse it to map, if failed then the return value is nil
    // A check for nil is always a good practise if different
    // kind of values or no data are registered via `ViewData`.
    //
    // Similarly to `viewData := ctx.Values().Get("iris.viewData")` or
    // `viewData := ctx.Values().Get(ctx.Application().ConfigurationReadOnly().GetViewDataContextKey())`.
    GetViewData() map[string]interface{}
    // View renders a template based on the registered view engine(s).
    // First argument accepts the filename, relative to the view engine's Directory and Extension,
    // i.e: if directory is "./templates" and want to render the "./templates/users/index.html"
    // then you pass the "users/index.html" as the filename argument.
    //
    // The second optional argument can receive a single "view model"
    // that will be binded to the view template if it's not nil,
    // otherwise it will check for previous view data stored by the `ViewData`
    // even if stored at any previous handler(middleware) for the same request.
    //
    // Look .ViewData` and .ViewLayout too.
    //
    // Examples: https://github.com/kataras/iris/tree/master/_examples/view
    View(filename string, optionalViewModel ...interface{}) error

    // Binary writes out the raw bytes as binary data.
    Binary(data []byte) (int, error)
    // Text writes out a string as plain text.
    Text(format string, args ...interface{}) (int, error)
    // HTML writes out a string as text/html.
    HTML(format string, args ...interface{}) (int, error)
    // JSON marshals the given interface object and writes the JSON response.
    JSON(v interface{}, options ...JSON) (int, error)
    // JSONP marshals the given interface object and writes the JSON response.
    JSONP(v interface{}, options ...JSONP) (int, error)
    // XML marshals the given interface object and writes the XML response.
    XML(v interface{}, options ...XML) (int, error)
    // Markdown parses the markdown to html and renders its result to the client.
    Markdown(markdownB []byte, options ...Markdown) (int, error)
    // YAML parses the "v" using the yaml parser and renders its result to the client.
    YAML(v interface{}) (int, error)
    //  +------------------------------------------------------------+
    //  | Serve files                                                |
    //  +------------------------------------------------------------+

    // ServeContent serves content, headers are autoset
    // receives three parameters, it's low-level function, instead you can use .ServeFile(string,bool)/SendFile(string,string)
    //
    //
    // You can define your own "Content-Type" with `context#ContentType`, before this function call.
    //
    // This function doesn't support resuming (by range),
    // use ctx.SendFile or router's `HandleDir` instead.
    ServeContent(content io.ReadSeeker, filename string, modtime time.Time, gzipCompression bool) error
    // ServeFile serves a file (to send a file, a zip for example to the client you should use the `SendFile` instead)
    // receives two parameters
    // filename/path (string)
    // gzipCompression (bool)
    //
    // You can define your own "Content-Type" with `context#ContentType`, before this function call.
    //
    // This function doesn't support resuming (by range),
    // use ctx.SendFile or router's `HandleDir` instead.
    //
    // Use it when you want to serve dynamic files to the client.
    ServeFile(filename string, gzipCompression bool) error
    // SendFile sends file for force-download to the client
    //
    // Use this instead of ServeFile to 'force-download' bigger files to the client.
    SendFile(filename string, destinationName string) error

    //  +------------------------------------------------------------+
    //  | Cookies                                                    |
    //  +------------------------------------------------------------+

    // SetCookie adds a cookie.
    // Use of the "options" is not required, they can be used to amend the "cookie".
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
    SetCookie(cookie *http.Cookie, options ...CookieOption)
    // SetCookieKV adds a cookie, requires the name(string) and the value(string).
    //
    // By default it expires at 2 hours and it's added to the root path,
    // use the `CookieExpires` and `CookiePath` to modify them.
    // Alternatively: ctx.SetCookie(&http.Cookie{...})
    //
    // If you want to set custom the path:
    // ctx.SetCookieKV(name, value, iris.CookiePath("/custom/path/cookie/will/be/stored"))
    //
    // If you want to be visible only to current request path:
    // ctx.SetCookieKV(name, value, iris.CookieCleanPath/iris.CookiePath(""))
    // More:
    //                              iris.CookieExpires(time.Duration)
    //                              iris.CookieHTTPOnly(false)
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
    SetCookieKV(name, value string, options ...CookieOption)
    // GetCookie returns cookie's value by its name
    // returns empty string if nothing was found.
    //
    // If you want more than the value then:
    // cookie, err := ctx.Request().Cookie("name")
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
    GetCookie(name string, options ...CookieOption) string
    // RemoveCookie deletes a cookie by its name and path = "/".
    // Tip: change the cookie's path to the current one by: RemoveCookie("name", iris.CookieCleanPath)
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
    RemoveCookie(name string, options ...CookieOption)
    // VisitAllCookies accepts a visitor function which is called
    // on each (request's) cookies' name and value.
    VisitAllCookies(visitor func(name string, value string))

    // MaxAge returns the "cache-control" request header's value
    // seconds as int64
    // if header not found or parse failed then it returns -1.
    MaxAge() int64

    //  +------------------------------------------------------------+
    //  | Advanced: Response Recorder and Transactions               |
    //  +------------------------------------------------------------+

    // Record transforms the context's basic and direct responseWriter to a ResponseRecorder
    // which can be used to reset the body, reset headers, get the body,
    // get & set the status code at any time and more.
    Record()
    // Recorder returns the context's ResponseRecorder
    // if not recording then it starts recording and returns the new context's ResponseRecorder
    Recorder() *ResponseRecorder
    // IsRecording returns the response recorder and a true value
    // when the response writer is recording the status code, body, headers and so on,
    // else returns nil and false.
    IsRecording() (*ResponseRecorder, bool)

    // BeginTransaction starts a scoped transaction.
    //
    // You can search third-party articles or books on how Business Transaction works (it's quite simple, especially here).
    //
    // Note that this is unique and new
    // (=I haver never seen any other examples or code in Golang on this subject, so far, as with the most of iris features...)
    // it's not covers all paths,
    // such as databases, this should be managed by the libraries you use to make your database connection,
    // this transaction scope is only for context's response.
    // Transactions have their own middleware ecosystem also, look iris.go:UseTransaction.
    //
    // See https://github.com/kataras/iris/tree/master/_examples/ for more
    BeginTransaction(pipe func(t *Transaction))
    // SkipTransactions if called then skip the rest of the transactions
    // or all of them if called before the first transaction
    SkipTransactions()
    // TransactionsSkipped returns true if the transactions skipped or canceled at all.
    TransactionsSkipped() bool

    // Exec calls the `context/Application#ServeCtx`
    // based on this context but with a changed method and path
    // like it was requested by the user, but it is not.
    //
    // Offline means that the route is registered to the iris and have all features that a normal route has
    // BUT it isn't available by browsing, its handlers executed only when other handler's context call them
    // it can validate paths, has sessions, path parameters and all.
    //
    // You can find the Route by app.GetRoute("theRouteName")
    // you can set a route name as: myRoute := app.Get("/mypath", handler)("theRouteName")
    // that will set a name to the route and returns its RouteInfo instance for further usage.
    //
    // It doesn't changes the global state, if a route was "offline" it remains offline.
    //
    // app.None(...) and app.GetRoutes().Offline(route)/.Online(route, method)
    //
    // Example: https://github.com/kataras/iris/tree/master/_examples/routing/route-state
    //
    // User can get the response by simple using rec := ctx.Recorder(); rec.Body()/rec.StatusCode()/rec.Header().
    //
    // Context's Values and the Session are kept in order to be able to communicate via the result route.
    //
    // It's for extreme use cases, 99% of the times will never be useful for you.
    Exec(method, path string)

    // RouteExists reports whether a particular route exists
    // It will search from the current subdomain of context's host, if not inside the root domain.
    RouteExists(method, path string) bool

    // Application returns the iris app instance which belongs to this context.
    // Worth to notice that this function returns an interface
    // of the Application, which contains methods that are safe
    // to be executed at serve-time. The full app's fields
    // and methods are not available here for the developer's safety.
    Application() Application

    // String returns the string representation of this request.
    // Each context has a unique string representation.
    // It can be used for simple debugging scenarios, i.e print context as string.
    //
    // What it returns? A number which declares the length of the
    // total `String` calls per executable application, followed
    // by the remote IP (the client) and finally the method:url.
    String() string
}
```
