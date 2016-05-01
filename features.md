# Features

* **Typescript**: Auto-compile & Watch your client side code via the [typescript plugin](https://github.com/kataras/iris/tree/development/plugin/typescript)
* **Online IDE**: Edit & Compile your client side code when you are not home via the [editor plugin](editor.md)
* **Iris Online Control**: Web-based interface to control the basics functionalities of your server via the [iriscontrol plugin](iriscontrol.md). Note that Iris control is still young
* **Subdomains**: Easy way to express your api via custom and dynamic subdomains[*](subdomains.md)
* **Named Path Parameters**: Probably you already know what that means. If not, [It's easy to learn about](named-parameters.md)
* **Custom HTTP Errors**: Define your own html templates or plain messages when http errors occurs[*](custom-http-errors.md)
* **Internationalization**: [i18n](internationalization-and-localization.md)
* **Bindings**: Need a fast way to convert data from body or form into an object? Take a look [here](https://github.com/iris-contrib/examples/tree/master/bind_form_simple)
* **Streaming**: You have only one option when streaming comes in game[*](streaming.md)
* **Middlewares**: Create and/or use global or per route middlewares with the Iris' simplicity[*](middlewares.md)
* **Sessions**:  Sessions provides a secure way to authenticate your clients/users [*](sessions.md)
* **Realtime**: Realtime is fun when you use websockets[*](websockets.md)
* **Context**: [Context](#context) is used for storing route params, storing handlers, sharing variables between middlewares, render rich content, send file and much more[*](context.md)
* **Plugins**: You can build your own plugins to  inject the Iris framework[*](plugins.md)
* **Full API**: All http methods are supported[*](api.md)
* **Party**:  Group routes when sharing the same resources or middlewares. You can organise a party with domains too! [*](party.md)
* **Transport Layer Security**: Provide privacy and data integrity between your server and the client[*](#tls)
* **Multi server instances**: Besides the fact that Iris has a default main server. You can declare as many as you need[*](declaration.md)
* **Zero allocations**: Iris generates zero garbage