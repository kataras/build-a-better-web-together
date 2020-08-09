# Context Methods

Here is a full list of methods that the `iris.Context` structure provides.

```go
// Clone returns a copy of the context that
// can be safely used outside the request's scope.
// Note that if the request-response lifecycle terminated
// or request canceled by the client (can be checked by `ctx.IsCanceled()`)
// then the response writer is totally useless.
// The http.Request pointer value is shared.
Clone() *Context
// BeginRequest is executing once for each request
// it should prepare the (new or acquired from pool) context's fields for the new request.
// Do NOT call it manually. Framework calls it automatically.
//
// Resets
// 1. handlers to nil.
// 2. values to empty.
// 3. the defer function.
// 4. response writer to the http.ResponseWriter.
// 5. request to the *http.Request.
BeginRequest(w http.ResponseWriter, r *http.Request)
// EndRequest is executing once after a response to the request was sent and this context is useless or released.
// Do NOT call it manually. Framework calls it automatically.
//
// 1. executes the OnClose function (if any).
// 2. flushes the response writer's result or fire any error handler.
// 3. releases the response writer.
EndRequest()
// IsCanceled reports whether the client canceled the request
// or the underlying connection has gone.
// Note that it will always return true
// when called from a goroutine after the request-response lifecycle.
IsCanceled() bool
// OnConnectionClose registers the "cb" Handler
// which will be fired on its on goroutine on a cloned Context
// when the underlying connection has gone away.
//
// The code inside the given callback is running on its own routine,
// as explained above, therefore the callback should NOT
// try to access to handler's Context response writer.
//
// This mechanism can be used to cancel long operations on the server
// if the client has disconnected before the response is ready.
//
// It depends on the Request's Context.Done() channel.
//
// Finally, it reports whether the protocol supports pipelines (HTTP/1.1 with pipelines disabled is not supported).
// The "cb" will not fire for sure if the output value is false.
//
// Note that you can register only one callback per route.
//
// See `OnClose` too.
OnConnectionClose(cb Handler) bool
// OnClose registers a callback which
// will be fired when the underlying connection has gone away(request canceled)
// on its own goroutine or in the end of the request-response lifecylce
// on the handler's routine itself (Context access).
//
// See `OnConnectionClose` too.
OnClose(cb Handler)
// ResponseWriter returns an http.ResponseWriter compatible response writer, as expected.
ResponseWriter() ResponseWriter
// ResetResponseWriter sets a new ResponseWriter implementation
// to this Context to use as its writer.
// Note, to change the underline http.ResponseWriter use
// ctx.ResponseWriter().SetWriter(http.ResponseWRiter) instead.
ResetResponseWriter(newResponseWriter ResponseWriter)
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
// SetCurrentRoute sets the route internally,
// See `GetCurrentRoute()` method too.
// It's being initialized by the Router.
// See `Exec` or `SetHandlers/AddHandler` methods to simulate a request.
SetCurrentRoute(route RouteReadOnly)
// GetCurrentRoute returns the current "read-only" route that
// was registered to this request's path.
GetCurrentRoute() RouteReadOnly
// Do sets the "handlers" as the chain
// and executes the first handler,
// handlers should not be empty.
//
// It's used by the router, developers may use that
// to replace and execute handlers immediately.
Do(handlers Handlers)
// AddHandler can add handler(s)
// to the current request in serve-time,
// these handlers are not persistenced to the router.
//
// Router is calling this function to add the route's handler.
// If AddHandler called then the handlers will be inserted
// to the end of the already-defined route's handler.
//
AddHandler(handlers ...Handler)
// SetHandlers replaces all handlers with the new.
SetHandlers(handlers Handlers)
// Handlers keeps tracking of the current handlers.
Handlers() Handlers
// HandlerIndex sets the current index of the
// current context's handlers chain.
// If n < 0 or the current handlers length is 0 then it just returns the
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
Proceed(h Handler) bool
// HandlerName returns the current handler's name, helpful for debugging.
HandlerName() string
// HandlerFileLine returns the current running handler's function source file and line information.
// Useful mostly when debugging.
HandlerFileLine() (file string, line int)
// RouteName returns the route name that this handler is running on.
// Note that it may return empty on not found handlers.
RouteName() string
// Next calls the next handler from the handlers chain,
// it should be used inside a middleware.
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
// StopExecution stops the handlers chain of this request.
// Meaning that any following `Next` calls are ignored,
// as a result the next handlers in the chain will not be fire.
StopExecution()
// IsStopped reports whether the current position of the context's handlers is -1,
// means that the StopExecution() was called at least once.
IsStopped() bool
// StopWithStatus stops the handlers chain and writes the "statusCode".
//
// If the status code is a failure one then
// it will also fire the specified error code handler.
StopWithStatus(statusCode int)
// StopWithText stops the handlers chain and writes the "statusCode"
// among with a message "plainText".
//
// If the status code is a failure one then
// it will also fire the specified error code handler.
StopWithText(statusCode int, plainText string)
// StopWithError stops the handlers chain and writes the "statusCode"
// among with the error "err".
// It Calls the `SetErr` method so error handlers can access the given error.
//
// If the status code is a failure one then
// it will also fire the specified error code handler.
StopWithError(statusCode int, err error)
// StopWithJSON stops the handlers chain, writes the status code
// and sends a JSON response.
//
// If the status code is a failure one then
// it will also fire the specified error code handler.
StopWithJSON(statusCode int, jsonObject interface{})
// StopWithProblem stops the handlers chain, writes the status code
// and sends an application/problem+json response.
// See `iris.NewProblem` to build a "problem" value correctly.
//
// If the status code is a failure one then
// it will also fire the specified error code handler.
StopWithProblem(statusCode int, problem Problem)
// Params returns the current url's named parameters key-value storage.
// Named path parameters are being saved here.
// This storage, as the whole context, is per-request lifetime.
Params() *RequestParams
// Values returns the current "user" storage.
// Named path parameters and any optional data can be saved here.
// This storage, as the whole context, is per-request lifetime.
//
// You can use this function to Set and Get local values
// that can be used to share information between handlers and middleware.
Values() *memstore.Store
// Method returns the request.Method, the client's http method to the server.
Method() string
// Path returns the full request path,
// escaped if EnablePathEscape config field is true.
Path() string
// RequestPath returns the full request path,
// based on the 'escape'.
RequestPath(escape bool) string
// Host returns the host part of the current URI.
// This method makes use of the `Configuration.HostProxyHeaders` field too.
Host() string
// Subdomain returns the subdomain of this request, if any.
// Note that this is a fast method which does not cover all cases.
Subdomain() (subdomain string)
// FindClosest returns a list of "n" paths close to
// this request based on subdomain and request path.
//
// Order may change.
// Example: https://github.com/kataras/iris/tree/master/_examples/routing/intelligence/manual
FindClosest(n int) []string
// IsWWW returns true if the current subdomain (if any) is www.
IsWWW() bool
// FullRequestURI returns the full URI,
// including the scheme, the host and the relative requested path/resource.
FullRequestURI() string
// RemoteAddr tries to parse and return the real client's request IP.
//
// Based on allowed headers names that can be modified from Configuration.RemoteAddrHeaders.
//
// If parse based on these headers fail then it will return the Request's `RemoteAddr` field
// which is filled by the server before the HTTP handler,
// unless the Configuration.RemoteAddrHeadersForce was set to true
// which will force this method to return the first IP from RemoteAddrHeaders
// even if it's part of a private network.
//
// Look `Configuration.RemoteAddrHeaders`,
//		`Configuration.RemoteAddrHeadersForce`,
//      `Configuration.WithRemoteAddrHeader(...)`,
//      `Configuration.WithoutRemoteAddrHeader(...)` and
//      `Configuration.RemoteAddrPrivateSubnets` for more.
RemoteAddr() string
// GetHeader returns the request header's value based on its name.
GetHeader(name string) string
// GetDomain resolves and returns the server's domain.
GetDomain() string
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
// IsScript reports whether a client is a script.
IsScript() bool
// IsSSL reports whether the client is running under HTTPS SSL.
//
// See `IsHTTP2` too.
IsSSL() bool
// IsHTTP2 reports whether the protocol version for incoming request was HTTP/2.
// The client code always uses either HTTP/1.1 or HTTP/2.
//
// See `IsSSL` too.
IsHTTP2() bool
// IsGRPC reports whether the request came from a gRPC client.
IsGRPC() bool
// GetReferrer extracts and returns the information from the "Referer" (or "Referrer") header
// and url query parameter as specified in https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Referrer-Policy.
GetReferrer() Referrer
// SetLanguage force-sets the language for i18n, can be used inside a middleare.
// It has the highest priority over the rest and if it is empty then it is ignored,
// if it set to a static string of "default" or to the default language's code
// then the rest of the language extractors will not be called at all and
// the default language will be set instead.
//
// See `i18n.ExtractFunc` for a more organised way of the same feature.
SetLanguage(langCode string)
// GetLocale returns the current request's `Locale` found by i18n middleware.
// See `Tr` too.
GetLocale() Locale
// Tr returns a i18n localized message based on format with optional arguments.
// See `GetLocale` too.
//
// Example: https://github.com/kataras/iris/tree/master/_examples/i18n
Tr(message string, values ...interface{}) string
// Header adds a header to the response, if value is empty
// it removes the header by its name.
Header(name string, value string)
// ContentType sets the response writer's
// header "Content-Type" to the 'cType'.
ContentType(cType string)
// GetContentType returns the response writer's
// header value of "Content-Type".
GetContentType() string
// GetContentTypeRequested returns the request's
// trim-ed(without the charset and priority values)
// header value of "Content-Type".
GetContentTypeRequested() string
// GetContentLength returns the request's
// header value of "Content-Length".
GetContentLength() int64
// StatusCode sets the status code header to the response.
// Look .GetStatusCode & .FireStatusCode too.
//
// Remember, the last one before .Write matters except recorder and transactions.
StatusCode(statusCode int)
// NotFound emits an error 404 to the client, using the specific custom error error handler.
// Note that you may need to call ctx.StopExecution() if you don't want the next handlers
// to be executed. Next handlers are being executed on iris because you can alt the
// error code and change it to a more specific one, i.e
// users := app.Party("/users")
// users.Done(func(ctx iris.Context){ if ctx.StatusCode() == 400 { /*  custom error code for /users */ }})
NotFound()
// GetStatusCode returns the current status code of the response.
// Look StatusCode too.
GetStatusCode() int
// URLParamExists returns true if the url parameter exists, otherwise false.
URLParamExists(name string) bool
// URLParamDefault returns the get parameter from a request, if not found then "def" is returned.
URLParamDefault(name string, def string) string
// URLParam returns the get parameter from a request, if any.
URLParam(name string) string
// URLParamTrim returns the url query parameter with trailing white spaces removed from a request.
URLParamTrim(name string) string
// URLParamEscape returns the escaped url query parameter from a request.
URLParamEscape(name string) string
// URLParamInt returns the url query parameter as int value from a request,
// returns -1 and an error if parse failed or not found.
URLParamInt(name string) (int, error)
// URLParamIntDefault returns the url query parameter as int value from a request,
// if not found or parse failed then "def" is returned.
URLParamIntDefault(name string, def int) int
// URLParamInt32Default returns the url query parameter as int32 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt32Default(name string, def int32) int32
// URLParamInt64 returns the url query parameter as int64 value from a request,
// returns -1 and an error if parse failed or not found.
URLParamInt64(name string) (int64, error)
// URLParamInt64Default returns the url query parameter as int64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamInt64Default(name string, def int64) int64
// URLParamFloat64 returns the url query parameter as float64 value from a request,
// returns an error and -1 if parse failed.
URLParamFloat64(name string) (float64, error)
// URLParamFloat64Default returns the url query parameter as float64 value from a request,
// if not found or parse failed then "def" is returned.
URLParamFloat64Default(name string, def float64) float64
// URLParamBool returns the url query parameter as boolean value from a request,
// returns an error if parse failed.
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
// If not found or parse errors returns the "def".
PostValueIntDefault(name string, def int) int
// PostValueInt64 returns the parsed form data from POST, PATCH,
// or PUT body parameters based on a "name", as float64.
//
// If not found returns -1 and a non-nil error.
PostValueInt64(name string) (int64, error)
// PostValueInt64Default returns the parsed form data from POST, PATCH,
// or PUT body parameters based on a "name", as int64.
//
// If not found or parse errors returns the "def".
PostValueInt64Default(name string, def int64) int64
// PostValueFloat64 returns the parsed form data from POST, PATCH,
// or PUT body parameters based on a "name", as float64.
//
// If not found returns -1 and a non-nil error.
PostValueFloat64(name string) (float64, error)
// PostValueFloat64Default returns the parsed form data from POST, PATCH,
// or PUT body parameters based on a "name", as float64.
//
// If not found or parse errors returns the "def".
PostValueFloat64Default(name string, def float64) float64
// PostValueBool returns the parsed form data from POST, PATCH,
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
//
// The default form's memory maximum size is 32MB, it can be changed by the
// `iris#WithPostMaxMemory` configurator at main configuration passed on `app.Run`'s second argument.
//
// Example: https://github.com/kataras/iris/tree/master/_examples/file-server/upload-file
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
// Example: https://github.com/kataras/iris/tree/master/_examples/file-server/upload-files
UploadFormFiles(destDirectory string, before ...func(*Context, *multipart.FileHeader)) (n int64, err error)
// AbsoluteURI parses the "s" and returns its absolute URI form.
AbsoluteURI(s string) string
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
// SetMaxRequestBodySize sets a limit to the request body size
// should be called before reading the request body from the client.
SetMaxRequestBodySize(limitOverBytes int64)
// GetBody reads and returns the request body.
// The default behavior for the http request reader is to consume the data readen
// but you can change that behavior by passing the `WithoutBodyConsumptionOnUnmarshal` iris option.
//
// However, whenever you can use the `ctx.Request().Body` instead.
GetBody() ([]byte, error)
// UnmarshalBody reads the request's body and binds it to a value or pointer of any type
// Examples of usage: context.ReadJSON, context.ReadXML.
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-custom-via-unmarshaler/main.go
UnmarshalBody(outPtr interface{}, unmarshaler Unmarshaler) error
// ReadJSON reads JSON from request's body and binds it to a value of any json-valid type.
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-json/main.go
ReadJSON(outPtr interface{}) error
// ReadXML reads XML from request's body and binds it to a value of any xml-valid type.
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-xml/main.go
ReadXML(outPtr interface{}) error
// ReadYAML reads YAML from request's body and binds it to the "outPtr" value.
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-yaml/main.go
ReadYAML(outPtr interface{}) error
// ReadForm binds the request body of a form to the "formObject".
// It supports any kind of type, including custom structs.
// It will return nothing if request data are empty.
// The struct field tag is "form".
// Note that it will return nil error on empty form data if `Configuration.FireEmptyFormError`
// is false (as defaulted) in this case the caller should check the pointer to
// see if something was actually binded.
//
// If a client sent an unknown field, this method will return an error,
// in order to ignore that error use the `err != nil && !iris.IsErrPath(err)`.
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-form/main.go
ReadForm(formObject interface{}) error
// ReadQuery binds url query to "ptr". The struct field tag is "url".
//
// Example: https://github.com/kataras/iris/blob/master/_examples/request-body/read-query/main.go
ReadQuery(ptr interface{}) error
// ReadProtobuf binds the body to the "ptr" of a proto Message and returns any error.
// Look `ReadJSONProtobuf` too.
ReadProtobuf(ptr proto.Message) error
// ReadJSONProtobuf reads a JSON body request into the given "ptr" proto.Message.
// Look `ReadProtobuf` too.
ReadJSONProtobuf(ptr proto.Message, opts ...ProtoUnmarshalOptions) error
// ReadMsgPack binds the request body of msgpack format to the "ptr" and returns any error.
ReadMsgPack(ptr interface{}) error
// ReadBody binds the request body to the "ptr" depending on the HTTP Method and the Request's Content-Type.
// If a GET method request then it reads from a form (or URL Query), otherwise
// it tries to match (depending on the request content-type) the data format e.g.
// JSON, Protobuf, MsgPack, XML, YAML, MultipartForm and binds the result to the "ptr".
ReadBody(ptr interface{}) error
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
Write(rawBody []byte) (int, error)
// Writef formats according to a format specifier and writes to the response.
//
// Returns the number of bytes written and any write error encountered.
Writef(format string, a ...interface{}) (n int, err error)
// WriteString writes a simple string to the response.
//
// Returns the number of bytes written and any write error encountered.
WriteString(body string) (n int, err error)
// SetLastModified sets the "Last-Modified" based on the "modtime" input.
// If "modtime" is zero then it does nothing.
//
// It's mostly internally on core/router and context packages.
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
// or if parsing time from the header failed. See `ErrPreconditionFailed` too.
//
// It's mostly used internally, e.g. `context#WriteWithExpiration`.
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
StreamWriter(writer func(w io.Writer) error) error
// ClientSupportsEncoding reports whether the
// client expects one of the given "encodings" compression.
//
// Note, this method just reports back the first valid encoding it sees,
// meaning that request accept-encoding offers don't matter here.
// See `CompressWriter` too.
ClientSupportsEncoding(encodings ...string) bool
// CompressWriter enables or disables the compress response writer.
// if the client expects a valid compression algorithm then this
// will change the response writer to a compress writer instead.
// All future write and rich write methods will respect this option.
// Usage:
// app.Use(func(ctx iris.Context){
// 	err := ctx.CompressWriter(true)
// 	ctx.Next()
// })
// The recommendation is to compress data as much as possible and therefore to use this field,
// but some types of resources, such as jpeg images, are already compressed.
// Sometimes, using additional compression doesn't reduce payload size and
// can even make the payload longer.
CompressWriter(enable bool) error
// CompressReader accepts a boolean, which, if set to true
// it wraps the request body reader with a reader which decompresses request data before read.
// If the "enable" input argument is false then the request body will reset to the default one.
//
// Useful when incoming request data are compressed.
// All future calls of `ctx.GetBody/ReadXXX/UnmarshalBody` methods will respect this option.
//
// Usage:
// app.Use(func(ctx iris.Context){
// 	err := ctx.CompressReader(true)
// 	ctx.Next()
// })
// More:
// if cr, ok := ctx.Request().Body.(*CompressReader); ok {
// 	cr.Src // the original request body
//  cr.Encoding // the compression algorithm.
// }
//
// It returns `ErrRequestNotCompressed` if client's request data are not compressed
// (or empty)
// or `ErrNotSupportedCompression` if server missing the decompression algorithm.
CompressReader(enable bool) error
// ViewLayout sets the "layout" option if and when .View
// is being called afterwards, in the same request.
// Useful when need to set or/and change a layout based on the previous handlers in the chain.
//
// Note that the 'layoutTmplFile' argument can be set to iris.NoLayout || view.NoLayout || context.NoLayout
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
// Look .ViewData and .ViewLayout too.
//
// Examples: https://github.com/kataras/iris/tree/master/_examples/view
View(filename string, optionalViewModel ...interface{}) error
// Binary writes out the raw bytes as binary data.
Binary(data []byte) (int, error)
// Text writes out a string as plain text.
Text(format string, args ...interface{}) (int, error)
// HTML writes out a string as text/html.
HTML(format string, args ...interface{}) (int, error)
// JSON marshals the given interface object and writes the JSON response to the client.
// If the value is a compatible `proto.Message` one
// then it only uses the options.Proto settings to marshal.
JSON(v interface{}, opts ...JSON) (n int, err error)
// JSONP marshals the given interface object and writes the JSON response to the client.
JSONP(v interface{}, opts ...JSONP) (int, error)
// XML marshals the given interface object and writes the XML response to the client.
// To render maps as XML see the `XMLMap` package-level function.
XML(v interface{}, opts ...XML) (int, error)
// Problem writes a JSON or XML problem response.
// Order of Problem fields are not always rendered the same.
//
// Behaves exactly like `Context.JSON`
// but with default ProblemOptions.JSON indent of " " and
// a response content type of "application/problem+json" instead.
//
// Use the options.RenderXML and XML fields to change this behavior and
// send a response of content type "application/problem+xml" instead.
//
// Read more at: https://github.com/kataras/iris/wiki/Routing-error-handlers
Problem(v interface{}, opts ...ProblemOptions) (int, error)
// Markdown parses the markdown to html and renders its result to the client.
Markdown(markdownB []byte, opts ...Markdown) (int, error)
// YAML marshals the "v" using the yaml marshaler and renders its result to the client.
YAML(v interface{}) (int, error)
// Protobuf parses the "v" of proto Message and renders its result to the client.
Protobuf(v proto.Message) (int, error)
// MsgPack parses the "v" of msgpack format and renders its result to the client.
MsgPack(v interface{}) (int, error)
// Negotiation creates once and returns the negotiation builder
// to build server-side available prioritized content
// for specific content type(s), charset(s) and encoding algorithm(s).
//
// See `Negotiate` method too.
Negotiation() *NegotiationBuilder
// Negotiate used for serving different representations of a resource at the same URI.
//
// The "v" can be a single `N` struct value.
// The "v" can be any value completes the `ContentSelector` interface.
// The "v" can be any value completes the `ContentNegotiator` interface.
// The "v" can be any value of struct(JSON, JSONP, XML, YAML, Protobuf, MsgPack) or
// string(TEXT, HTML) or []byte(Markdown, Binary) or []byte with any matched mime type.
//
// If the "v" is nil, the `Context.Negotitation()` builder's
// content will be used instead, otherwise "v" overrides builder's content
// (server mime types are still retrieved by its registered, supported, mime list)
//
// Set mime type priorities by `Negotiation().JSON().XML().HTML()...`.
// Set charset priorities by `Negotiation().Charset(...)`.
// Set encoding algorithm priorities by `Negotiation().Encoding(...)`.
// Modify the accepted by
// `Negotiation().Accept./Override()/.XML().JSON().Charset(...).Encoding(...)...`.
//
// It returns `ErrContentNotSupported` when not matched mime type(s).
//
// Resources:
// https://developer.mozilla.org/en-US/docs/Web/HTTP/Content_negotiation
// https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept
// https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Charset
// https://developer.mozilla.org/en-US/docs/Web/HTTP/Headers/Accept-Encoding
//
// Supports the above without quality values.
//
// Read more at: https://github.com/kataras/iris/wiki/Content-negotiation
Negotiate(v interface{}) (int, error)
// ServeContent replies to the request using the content in the
// provided ReadSeeker. The main benefit of ServeContent over io.Copy
// is that it handles Range requests properly, sets the MIME type, and
// handles If-Match, If-Unmodified-Since, If-None-Match, If-Modified-Since,
// and If-Range requests.
//
// If the response's Content-Type header is not set, ServeContent
// first tries to deduce the type from name's file extension.
//
// The name is otherwise unused; in particular it can be empty and is
// never sent in the response.
//
// If modtime is not the zero time or Unix epoch, ServeContent
// includes it in a Last-Modified header in the response. If the
// request includes an If-Modified-Since header, ServeContent uses
// modtime to decide whether the content needs to be sent at all.
//
// The content's Seek method must work: ServeContent uses
// a seek to the end of the content to determine its size.
//
// If the caller has set w's ETag header formatted per RFC 7232, section 2.3,
// ServeContent uses it to handle requests using If-Match, If-None-Match, or If-Range.
//
// Note that *os.File implements the io.ReadSeeker interface.
// Note that compression can be registered
// through `ctx.CompressWriter(true)` or `app.Use(iris.Compression)`.
ServeContent(content io.ReadSeeker, filename string, modtime time.Time)
// ServeContentWithRate same as `ServeContent` but it can throttle the speed of reading
// and though writing the "content" to the client.
ServeContentWithRate(content io.ReadSeeker, filename string, modtime time.Time, limit float64, burst int)
// ServeFile replies to the request with the contents of the named
// file or directory.
//
// If the provided file or directory name is a relative path, it is
// interpreted relative to the current directory and may ascend to
// parent directories. If the provided name is constructed from user
// input, it should be sanitized before calling `ServeFile`.
//
// Use it when you want to serve assets like css and javascript files.
// If client should confirm and save the file use the `SendFile` instead.
// Note that compression can be registered
// through `ctx.CompressWriter(true)` or `app.Use(iris.Compression)`.
ServeFile(filename string) error
// ServeFileWithRate same as `ServeFile` but it can throttle the speed of reading
// and though writing the file to the client.
ServeFileWithRate(filename string, limit float64, burst int) error
// SendFile sends a file as an attachment, that is downloaded and saved locally from client.
// Note that compression can be registered
// through `ctx.CompressWriter(true)` or `app.Use(iris.Compression)`.
// Use `ServeFile` if a file should be served as a page asset instead.
SendFile(src string, destName string) error
// SendFileWithRate same as `SendFile` but it can throttle the speed of reading
// and though writing the file to the client.
SendFileWithRate(src, destName string, limit float64, burst int) error
// AddCookieOptions adds cookie options for `SetCookie`,
// `SetCookieKV, UpsertCookie` and `RemoveCookie` methods
// for the current request. It can be called from a middleware before
// cookies sent or received from the next Handler in the chain.
//
// Available builtin Cookie options are:
//  * CookieAllowReclaim
//  * CookieAllowSubdomains
//  * CookieSecure
//  * CookieHTTPOnly
//  * CookieSameSite
//  * CookiePath
//  * CookieCleanPath
//  * CookieExpires
//  * CookieEncoding
//
// Example at: https://github.com/kataras/iris/tree/master/_examples/cookies/securecookie
AddCookieOptions(options ...CookieOption)
// ClearCookieOptions clears any previously registered cookie options.
// See `AddCookieOptions` too.
ClearCookieOptions()
// SetCookie adds a cookie.
// Use of the "options" is not required, they can be used to amend the "cookie".
//
// Example: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
SetCookie(cookie *http.Cookie, options ...CookieOption)
// UpsertCookie adds a cookie to the response like `SetCookie` does
// but it will also perform a replacement of the cookie
// if already set by a previous `SetCookie` call.
// It reports whether the cookie is new (true) or an existing one was updated (false).
UpsertCookie(cookie *http.Cookie, options ...CookieOption) bool
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
// (note that client should be responsible for that if server sent an empty cookie's path, all browsers are compatible)
// ctx.SetCookieKV(name, value, iris.CookieCleanPath/iris.CookiePath(""))
// More:
//                              iris.CookieExpires(time.Duration)
//                              iris.CookieHTTPOnly(false)
//
// Examples: https://github.com/kataras/iris/tree/master/_examples/cookies/basic
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
// VisitAllCookies takes a visitor function which is called
// on each (request's) cookies' name and value.
VisitAllCookies(visitor func(name string, value string))
// MaxAge returns the "cache-control" request header's value
// seconds as int64
// if header not found or parse failed then it returns -1.
MaxAge() int64
// Record transforms the context's basic and direct responseWriter to a *ResponseRecorder
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
// Can't say a lot here because it will take more than 200 lines to write about.
// You can search third-party articles or books on how Business Transaction works (it's quite simple, especially here).
//
// Note that this is unique and new
// (=I haver never seen any other examples or code in Golang on this subject, so far, as with the most of iris features...)
// it's not covers all paths,
// such as databases, this should be managed by the libraries you use to make your database connection,
// this transaction scope is only for context's response.
// Transactions have their own middleware ecosystem also.
//
// See https://github.com/kataras/iris/tree/master/_examples/ for more
BeginTransaction(pipe func(t *Transaction))
// SkipTransactions if called then skip the rest of the transactions
// or all of them if called before the first transaction
SkipTransactions()
// TransactionsSkipped returns true if the transactions skipped or canceled at all.
TransactionsSkipped() bool
// Exec calls the framewrok's ServeHTTPC
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
// context's Values and the Session are kept in order to be able to communicate via the result route.
//
// It's for extreme use cases, 99% of the times will never be useful for you.
Exec(method string, path string)
// RouteExists reports whether a particular route exists
// It will search from the current subdomain of context's host, if not inside the root domain.
RouteExists(method, path string) bool
// ReflectValue caches and returns a []reflect.Value{reflect.ValueOf(ctx)}.
// It's just a helper to maintain variable inside the context itself.
ReflectValue() []reflect.Value
// Controller returns a reflect Value of the custom Controller from which this handler executed.
// It will return a Kind() == reflect.Invalid if the handler was not executed from within a controller.
Controller() reflect.Value
// RegisterDependency registers a struct or slice
// or pointer to struct dependency at request-time
// for the next handler in the chain. One value per type.
// Note that it's highly recommended to register
// your dependencies before server ran
// through Party.ConfigureContainer or mvc.Application.Register
// in sake of minimum performance cost.
//
// See `UnregisterDependency` too.
RegisterDependency(v interface{})
// UnregisterDependency removes a dependency based on its type.
// Reports whether a dependency with that type was found and removed successfully.
UnregisterDependency(typ reflect.Type) bool
// Application returns the iris app instance which belongs to this context.
// Worth to notice that this function returns an interface
// of the Application, which contains methods that are safe
// to be executed at serve-time. The full app's fields
// and methods are not available here for the developer's safety.
Application() Application
// SetErr is just a helper that sets an error value
// as a context value, it does nothing more.
// Also, by-default this error's value is written to the client
// on failures when no registered error handler is available (see `Party.On(Any)ErrorCode`).
// See `GetError` to retrieve it back.
//
// Note that, if you want to stop the chain
// with an error see the `StopWithError` instead.
SetErr(err error)
```
<!-- slide:break-80 -->
