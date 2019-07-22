# Automatic Public Domain with TLS

Wouldn't be great to test your web application server in a more "real-world environment" like a public, remote, address instead of localhost?

There are plenty of third-party tools offering such a feature, but in my opinion, the [ngrok](https://github.com/inconshreveable/ngrok) one is the best among them. It's popular and tested for years, like Iris.

Iris offers ngrok integration. This feature is simple yet very powerful. It really helps when you want to quickly show your development progress to your colleagues or the project leader at a remote conference.

Follow the steps below to, temporarily, convert your local Iris web server to a public one.

1. Go head and [download ngrok](https://ngrok.io), add it to your $PATH environment variable,
2. Simply pass the `WithTunneling` configurator in your `app.Run`,
3. You are ready to [GO](https://www.facebook.com/iris.framework/photos/a.2420499271295384/3261189020559734/?type=3&theater)!

[![tunneling/_screenshot](https://user-images.githubusercontent.com/22900943/61413905-50596300-a8f5-11e9-8be0-7e806846d52f.png)](https://www.facebook.com/iris.framework/photos/a.2420499271295384/3261189020559734/?type=3&theater)

* `ctx.Application().ConfigurationReadOnly().GetVHost()` returns the public domain value. Rarely useful but it's there for you. Most of the times you use relative url paths instead of absolute\(or you should to\).
* It doesn't matter if ngrok is already running or not, Iris framework is smart enough to use ngrok's [web API](https://ngrok.com/docs) to create a tunnel.

Full `Tunneling` configuration:

```go
app.Run(iris.Addr(":8080"), iris.WithConfiguration(
    iris.Configuration{
        Tunneling: iris.TunnelingConfiguration{
            AuthToken:    "my-ngrok-auth-client-token",
            Bin:          "/bin/path/for/ngrok",
            Region:       "eu",
            WebInterface: "127.0.0.1:4040",
            Tunnels: []iris.Tunnel{
                {
                    Name: "MyApp",
                    Addr: ":8080",
                },
            },
        },
}))
```

Read more about [Configuration](../configuration.md).

