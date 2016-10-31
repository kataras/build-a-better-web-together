# Render

Think of 'Render' as an action which sends/responds with rich content to the client.

The render actions are separated into two categories:

* **Responses** send content using `Serialize Engines` which use the `Content-Type` as a `Key (explained later)`. (i.e. JSON, XML etc.)

* **Templates** send content using `Template Engines` which use file name extensions. (i.e. Markdown, Jade etc.)


### [Serialize Engines](/serialize-engines.md)

Easy and fast way to render any type of data. **JSON, JSONP, XML, Text, Data, Markdown** or any custom type. 

- examples are located [here](https://github.com/iris-contrib/examples/tree/4.0.0/serialize_engines/).

### [Template Engines](/template-engines.md)

Iris gives you the freedom to render templates through 6+ built-in template engines, 
you can create your own and 'inject' it to the iris station. 
You can also use more than one template engines at the same time (when the file extensions are different from each other). 

- examples are located [here](https://github.com/iris-contrib/examples/tree/4.0.0/template_engines/).

