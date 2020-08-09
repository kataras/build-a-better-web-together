# Protocol Buffers

**Content-Type: "application/x-protobuf"**

Protocol buffers are Google's language-neutral, platform-neutral, extensible mechanism for serializing structured data – think XML, but smaller, faster, and simpler. You define how you want your data to be structured once, then you can use special generated source code to easily write and read your structured data to and from a variety of data streams and using a variety of languages. https://developers.google.com/protocol-buffers

The `Context.Protobuf(proto.Message)` is the method which sends protos to the client. It accepts a [proto.Message](https://godoc.org/google.golang.org/protobuf/proto#Message) value.

> Note: Iris is using the newest version of the Go protocol buffers implementation. Read more about it at [The Go Blog: A new Go API for Protocol Buffers](https://blog.golang.org/protobuf-apiv2).


The methods you need to know when working with Protocol Buffers are the following:

```go
// Protobuf writes a proto message to the client,
// which should be able to read and parse protos too. 
Protobuf(v proto.Message) (int, error)
// ReadProtobuf binds the request body
// to the proto message.
ReadProtobuf(ptr proto.Message) error
```

```go
// JSON renders a proto.Message compatible value as JSON.
JSON(v interface{}, options ...JSON) (int, error)
// ReadJSONProtobuf reads a JSON body request
// into the given "ptr" proto.Message.
ReadJSONProtobuf(ptr proto.Message, opts ...protojson.UnmarshalOptions) error
```

```go
type JSON struct {
    // [...other fields]
	Proto ProtoMarshalOptions
}

type ProtoMarshalOptions struct {
    // Multiline specifies whether the marshaler
    // should format the output in
    // indented-form with every textual element
    // on a new line.
    // If Indent is an empty string,
    // then an arbitrary indent is chosen.
	Multiline bool

    // Indent specifies the set of indentation
    // characters to use in a multiline
    // formatted output such that every entry
    // is preceded by Indent and
    // terminated by a newline. If non-empty,
    // then Multiline is treated as true.
	// Indent can only be composed of space or tab characters.
	Indent string

    // AllowPartial allows messages that have
    // missing required fields to marshal
    // without returning an error.
    // If AllowPartial is false (the default),
    // Marshal will return error if there are
    // any missing required fields.
	AllowPartial bool

    // UseProtoNames uses proto field name
    // instead of lowerCamelCase name in JSON
	// field names.
	UseProtoNames bool

	// UseEnumNumbers emits enum values as numbers.
	UseEnumNumbers bool

    // EmitUnpopulated specifies whether to emit unpopulated fields.
    // It does not emit unpopulated oneof fields
    // or unpopulated extension fields.
	// The JSON value emitted for unpopulated fields are as follows:
	// ╔═══════╤════════════════════════════
	// ║ JSON  │ Protobuf field             
	// ╠═══════╪════════════════════════════
	// ║ false │ proto3 boolean fields      
	// ║ 0     │ proto3 numeric fields      
	// ║ ""    │ proto3 string/bytes fields 
	// ║ null  │ proto2 scalar fields       
	// ║ null  │ message fields             
	// ║ []    │ list fields                
	// ║ {}    │ map fields                 
	// ╚═══════╧════════════════════════════
	EmitUnpopulated bool
}
```

## Example

Let's create a simple application that showcases the use case of all of the above methods.

```text
│   main.go
│   go.mod
│   go.sum
└───protos
│      hello.proto
│      hello.pb.go
```

Navigate to the project's directory, create your go module: `$ go mod init app`.

The `protos/hello.proto` contents should look like that:

```proto
syntax = "proto3";

package protos;

option go_package = "./protos";

message HelloRequest {
    string name = 1;
}

message HelloReply {
    string message = 1;
}
```

To generate the `protos/hello.pb.go` file follow the commands below:

1. Install the protoc-gen-go tool.

```sh
$ go get -u google.golang.org/protobuf/cmd/protoc-gen-go@latest
```

2. Generate proto

```sh
$ protoc -I protos/ protos/hello.proto --go_out=.
```

Now, create the `main.go` file and copy-paste the following code:

```go
package main

import (
	"app/protos"

	"github.com/kataras/iris/v12"
)

func main() {
	app := iris.New()

	app.Get("/", send)
	app.Get("/json", sendAsJSON)
	app.Post("/read", read)
	app.Post("/read_json", readFromJSON)

	app.Listen(":8080")
}

func send(ctx iris.Context) {
	response := &protos.HelloReply{Message: "Hello, World!"}
	ctx.Protobuf(response)
}

func sendAsJSON(ctx iris.Context) {
	response := &protos.HelloReply{Message: "Hello, World!"}
	options := iris.JSON{
		Proto: iris.ProtoMarshalOptions{
			AllowPartial: true,
			Multiline:    true,
			Indent:       "    ",
		},
	}

	ctx.JSON(response, options)
}

func read(ctx iris.Context) {
	var request protos.HelloRequest

	err := ctx.ReadProtobuf(&request)
	if err != nil {
		ctx.StopWithError(iris.StatusBadRequest, err)
		return
	}

	ctx.Writef("HelloRequest.Name = %s", request.Name)
}

func readFromJSON(ctx iris.Context) {
	var request protos.HelloRequest

	err := ctx.ReadJSONProtobuf(&request)
	if err != nil {
		ctx.StopWithError(iris.StatusBadRequest, err)
		return
	}

	ctx.Writef("HelloRequest.Name = %s", request.Name)
}
```

> If you want to learn about Iris gRPC support, please navigate through the [mvc/grpc](../mvc/mvc-grpc.md) section instead.
<!-- slide:break-80 -->
