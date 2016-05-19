# Configuration

Configuration owns the relative package `github.com/kataras/iris/config` 

>  No need to download it separately, it's being downloaded automatically when you installed Iris.

### Why?
I took this decision after a lot of thought and I ensure you that this is the best
architecture to easy:

- change the configs without need to re-write all of their fields.
	```go
	irisConfig := config.Iris { Profile: true, PathCorrection: false }
	api := iris.New(irisConfig)
	```
- **easy to remember**: `iris` type takes `config.Iris`, sessions takes `config.Sessions`, `iris.Config().Render` is the `config.Render`, `iris.Config().Render.Template` is the `config.Template`, `Logger` takes `config.Logger` and so on...

- **easy to search & find out what features are exists and what you can change**: just navigate to the config folder and open the type you want to learn about, for example `/iris.go` Iris' type configuration is on `/config/iris.go`

- **All structs which receives configuration are already default-setted** , so don't worry too much, but if you ever need them you can find their default configs by this pattern: for example `config.Template` has `config.DefaultTemplate()`, `config.Rest` has `config.DefaultRest()`, `config.Typescript()` has `config.DefaultTypescript()`, note that only `config.Iris` has `config.Default()`. Eeven the plugins have their default configs, to make it easier for you. 

- so you can do this **without pre-set a config by yourself**: `iris.Config().Render.Template.Engine = config.PongoEngine` or `iris.Config().Render.Template.Pongo.Extensions = []string{".xhtml", ".html"}`.

- **(Advanced usage) merge configs**: 

```go
//...
import "github.com/kataras/iris/config"
//...
templateFromRoutine1 := config.DefaultTemplate()
//....
templateFromOthers := config.Template{ Directory: "views"}

templateConfig := templateFromRoutine1.MergeSingle(templateFromOthers)

iris.Config().Render.Template = templateConfig

```

Below you will find a list of the config structs.

## Search [All Configs](https://github.com/kataras/iris/tree/master/config)
```go
type (
	// Iris configs for the station
	// All fields can be changed before server's listen except the PathCorrection field
	//
	// MaxRequestBodySize is the only options that can be changed after server listen -
	// using Config().MaxRequestBodySize = ...
	// Render's rest config can be changed after declaration but before server's listen -
	// using Config().Render.Rest...
	// Render's Template config can be changed after declaration but before server's listen -
	// using Config().Render.Template...
	// Sessions config can be changed after declaration but before server's listen -
	// using Config().Sessions...
	// and so on...
	Iris struct {
		// MaxRequestBodySize Maximum request body size.
		//
		// The server rejects requests with bodies exceeding this limit.
		//
		// By default request body size is unlimited.
		MaxRequestBodySize int
		// PathCorrection corrects and redirects the requested path to the registed path
		// for example, if /home/ path is requested but no handler for this Route found,
		// then the Router checks if /home handler exists, if yes,
		// (permant)redirects the client to the correct path /home
		//
		// Default is true
		PathCorrection bool

		// Log turn it to false if you want to disable logger,
		// Iris prints/logs ONLY errors, so be careful when you disable it
		Log bool

		// Profile set to true to enable web pprof (debug profiling)
		// Default is false, enabling makes available these 7 routes:
		// /debug/pprof/cmdline
		// /debug/pprof/profile
		// /debug/pprof/symbol
		// /debug/pprof/goroutine
		// /debug/pprof/heap
		// /debug/pprof/threadcreate
		// /debug/pprof/pprof/block
		Profile bool

		// ProfilePath change it if you want other url path than the default
		// Default is /debug/pprof , which means yourhost.com/debug/pprof
		ProfilePath string

		// Sessions the config for sessions
		// contains 3(three) properties
		// Provider: (look /sessions/providers)
		// Secret: cookie's name (string)
		// Life: cookie life (time.Duration)
		Sessions Sessions

		// Render contains the configs for template and rest configuration
		Render Render
	}

	// Render struct keeps organise all configuration about rendering, templates and rest currently.
	Render struct {
		// Template the configs for template
		Template Template
		// Rest configs for rendering.
		//
		// these options inside this config don't have any relation with the TemplateEngine
		// from github.com/kataras/iris/rest
		Rest Rest
	}
)

```

```go
type (
	// Rest is a struct for specifying configuration options for the rest.Render object.
	Rest struct {
		// Appends the given character set to the Content-Type header. Default is "UTF-8".
		Charset string
		// Gzip enable it if you want to render with gzip compression. Default is false
		Gzip bool
		// Outputs human readable JSON.
		IndentJSON bool
		// Outputs human readable XML. Default is false.
		IndentXML bool
		// Prefixes the JSON output with the given bytes. Default is false.
		PrefixJSON []byte
		// Prefixes the XML output with the given bytes.
		PrefixXML []byte
		// Unescape HTML characters "&<>" to their original values. Default is false.
		UnEscapeHTML bool
		// Streams JSON responses instead of marshalling prior to sending. Default is false.
		StreamingJSON bool
		// Disables automatic rendering of http.StatusInternalServerError when an error occurs. 
        // Default is false.
		DisableHTTPErrorRendering bool
        // MarkdownSanitize sanitizes the markdown. Default is false.
		MarkdownSanitize bool
	}

	EngineType uint8

		Template struct {
		// contains common configs for both HTMLTemplate & Pongo
		Engine EngineType
		Gzip   bool
		IsDevelopment bool
		Directory     string
		Extensions    []string
		ContentType   string
		Charset       string
		Asset         func(name string) ([]byte, error)
		AssetNames    func() []string
		Layout        string

		HTMLTemplate HTMLTemplate // contains specific configs for  HTMLTemplate standard html/template
		Pongo        Pongo        // contains specific configs for pongo2
		// Markdown template engine it doesn't supports Layout & binding context
		Markdown Markdown // contains specific configs for markdown
		Jade     Jade     // contains specific configs for Jade
		Amber    Amber    // contains specific configs for Amber
	}

	HTMLTemplate struct {
		RequirePartials bool
		// Delims
		Left  string
		Right string
		// Funcs for HTMLTemplate html/template
		Funcs template.FuncMap
	}

	Pongo struct {
		// Filters for pongo2, map[name of the filter] the filter function . The filters are auto register
		Filters map[string]pongo2.FilterFunction
	}

	Markdown struct {
		Sanitize bool // if true then returns safe html, default is false
	}

	// Jade empty for now
	// stay tuned
	Jade struct {
	}

	Amber struct {
		// Funcs for the html/template result,
        // amber default funcs are not overrided so use it without worries
		Funcs template.FuncMap
	}
)
```

```go

var (
	universe time.Time // 0001-01-01 00:00:00 +0000 UTC
	// CookieExpireNever the default cookie's life for sessions, unlimited
	CookieExpireNever = universe
)

const (
	// DefaultCookieName the secret cookie's name for sessions
	DefaultCookieName        = "irissessionid"
	DefaultSessionGcDuration = time.Duration(2) * time.Hour
	// DefaultRedisNetwork the redis network option, "tcp"
	DefaultRedisNetwork = "tcp"
	// DefaultRedisAddr the redis address option, "127.0.0.1:6379"
	DefaultRedisAddr = "127.0.0.1:6379"
	// DefaultRedisIdleTimeout the redis idle timeout option, time.Duration(5) * time.Minute
	DefaultRedisIdleTimeout = time.Duration(5) * time.Minute
	// DefaultRedisMaxAgeSeconds the redis storage last parameter (SETEX), 31556926.0 (1 year)
	DefaultRedisMaxAgeSeconds = 31556926.0 //1 year

)

type (

	// Redis the redis configuration used inside sessions
	Redis struct {
		// Network "tcp"
		Network string
		// Addr "127.0.01:6379"
		Addr string
		// Password string .If no password then no 'AUTH'. Default ""
		Password string
		// If Database is empty "" then no 'SELECT'. Default ""
		Database string
		// MaxIdle 0 no limit
		MaxIdle int
		// MaxActive 0 no limit
		MaxActive int
		// IdleTimeout  time.Duration(5) * time.Minute
		IdleTimeout time.Duration
		// Prefix "myprefix-for-this-website". Default ""
		Prefix string
		// MaxAgeSeconds how much long the redis should keep the session in seconds. 
        // Default 31556926.0 (1 year)
		MaxAgeSeconds int
	}

	// Sessions the configuration for sessions
	// has 4 fields
	// first is the providerName (string) ["memory","redis"]
	// second is the cookieName, the session's name (string) ["mysessionsecretcookieid"]
	// third is the time which the client's cookie expires
	// forth is the gcDuration (time.Duration) 
    // when this time passes it removes the unused sessions from the memory until the user come back
	Sessions struct {
		// Provider string, usage iris.Config().Provider = "memory" or "redis".
        // If you wan to customize redis then import the package, and change it's config
		Provider string
		// Cookie string, the session's client cookie name, for example: "irissessionid"
		Cookie string
		//Expires the date which the cookie must expires. Default infinitive/unlimited life
		Expires time.Time
		//GcDuration every how much duration(GcDuration) 
        // the memory should be clear for unused cookies (GcDuration)
		//for example: time.Duration(2)*time.Hour. 
        // it will check every 2 hours if cookie hasn't be used for 2 hours,
		// deletes it from memory until the user comes back, 
        // then the session continue to work as it was
		//
		// Default 2 hours
		GcDuration time.Duration
	}
)
```

```go
type (
	Logger struct {
		Out    io.Writer
		Prefix string
		Flag   int
	}
)
```

```go
type (
	// Tsconfig the struct for tsconfig.json
	Tsconfig struct {
		CompilerOptions CompilerOptions `json:"compilerOptions"`
		Exclude         []string        `json:"exclude"`
	}

	// CompilerOptions contains all the compiler options used by the tsc (typescript compiler)
	CompilerOptions struct {
		Declaration                      bool   `json:"declaration"`
		Module                           string `json:"module"`
		Target                           string `json:"target"`
		Watch                            bool   `json:"watch"`
		Charset                          string `json:"charset"`
		Diagnostics                      bool   `json:"diagnostics"`
		EmitBOM                          bool   `json:"emitBOM"`
		EmitDecoratorMetadata            bool   `json:"emitDecoratorMetadata"`
		ExperimentalDecorators           bool   `json:"experimentalDecorators"`
		InlineSourceMap                  bool   `json:"inlineSourceMap"`
		InlineSources                    bool   `json:"inlineSources"`
		IsolatedModules                  bool   `json:"isolatedModules"`
		Jsx                              string `json:"jsx"`
		ReactNamespace                   string `json:"reactNamespace"`
		ListFiles                        bool   `json:"listFiles"`
		Locale                           string `json:"locale"`
		MapRoot                          string `json:"mapRoot"`
		ModuleResolution                 string `json:"moduleResolution"`
		NewLine                          string `json:"newLine"`
		NoEmit                           bool   `json:"noEmit"`
		NoEmitOnError                    bool   `json:"noEmitOnError"`
		NoEmitHelpers                    bool   `json:"noEmitHelpers"`
		NoImplicitAny                    bool   `json:"noImplicitAny"`
		NoLib                            bool   `json:"noLib"`
		NoResolve                        bool   `json:"noResolve"`
		SkipDefaultLibCheck              bool   `json:"skipDefaultLibCheck"`
		OutDir                           string `json:"outDir"`
		OutFile                          string `json:"outFile"`
		PreserveConstEnums               bool   `json:"preserveConstEnums"`
		Pretty                           bool   `json:"pretty"`
		RemoveComments                   bool   `json:"removeComments"`
		RootDir                          string `json:"rootDir"`
		SourceMap                        bool   `json:"sourceMap"`
		SourceRoot                       string `json:"sourceRoot"`
		StripInternal                    bool   `json:"stripInternal"`
		SuppressExcessPropertyErrors     bool   `json:"suppressExcessPropertyErrors"`
		SuppressImplicitAnyIndexErrors   bool   `json:"suppressImplicitAnyIndexErrors"`
		AllowUnusedLabels                bool   `json:"allowUnusedLabels"`
		NoImplicitReturns                bool   `json:"noImplicitReturns"`
		NoFallthroughCasesInSwitch       bool   `json:"noFallthroughCasesInSwitch"`
		AllowUnreachableCode             bool   `json:"allowUnreachableCode"`
		ForceConsistentCasingInFileNames bool   `json:"forceConsistentCasingInFileNames"`
		AllowSyntheticDefaultImports     bool   `json:"allowSyntheticDefaultImports"`
		AllowJs                          bool   `json:"allowJs"`
		NoImplicitUseStrict              bool   `json:"noImplicitUseStrict"`
	}

	Typescript struct {
		Bin      string
		Dir      string
		Ignore   string
		Tsconfig Tsconfig
		Editor   Editor
	}
)

```

```go

var (
	// DefaultUsername used for default (basic auth)
    // username in IrisControl's & Editor's default configuration
	DefaultUsername = "iris"
	// DefaultPassword used for default (basic auth) 
    // password in IrisControl's & Editor's default configuration
	DefaultPassword = "admin!123"
)

// IrisControl the options which iris control needs
// contains the port (int) and authenticated users with their passwords (map[string]string)
type IrisControl struct {
	// Port the port
	Port int
	// Users the authenticated users, [username]password
	Users map[string]string
}
```


```go
type Editor struct {
	// Host if empty used the iris server's host
	Host string
	// Port if 0 4444
	Port int
	// WorkingDir if empty "./"
	WorkingDir string
	// Useranme if empty iris
	Username string
	// Password if empty admin!123
	Password string
}
```