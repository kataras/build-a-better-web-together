# Body binder

Body binder reads values from the body and set them to a specific object.

```go
// ReadJSON reads JSON from request's body
ReadJSON(jsonObject interface{}) error

// ReadXML reads XML from request's body
ReadXML(xmlObject interface{}) error

// ReadForm binds the formObject  to the requeste's form data
func (ctx *Context) ReadForm(formObject interface{}) error
```

How to use

### JSON

```go
package main

import "github.com/kataras/iris"

type Company struct {
   Public     bool      `form:"public"`
   Website    url.URL   `form:"website"`
   Foundation time.Time `form:"foundation"`
   Name       string
   Location   struct {
     Country  string
     City     string
   }
   Products   []struct {
     Name string
     Type string
   }
   Founders   []string
   Employees  int64
}

func MyHandler(c *iris.Context) {
  if err := c.ReadJSON(&Company{}); err != nil {
  	panic(err.Error())
  }
}
  
func main() {
  iris.Get("/bind_json", MyHandler)
  iris.Listen(":8080")
}

```

### XML

```go
package main

import "github.com/kataras/iris"

type Company struct {
   Public     bool      
   Website    url.URL   
   Foundation time.Time
   Name       string
   Location   struct {
     Country  string
     City     string
   }
   Products   []struct {
     Name string
     Type string
   }
   Founders   []string
   Employees  int64
}
  
func MyHandler(c *iris.Context) {  
  if err := c.ReadXML(&Company{}); err != nil {
  	panic(err.Error())
  }
}
  
func main() {
  iris.Get("/bind_xml", MyHandler)
  iris.Listen(":8080")
}

```

### Form

#### Types

The supported field types in the destination struct are:

* `string`
* `bool`
* `int`, `int8`, `int16`, `int32`, `int64`
* `uint`, `uint8`, `uint16`, `uint32`, `uint64`
* `float32`, `float64`
* `slice`, `array`
* `struct` and `struct anonymous`
* `map`
* `interface{}`
* `time.Time`
* `url.URL`
*  slices []string
* `custom types` to one of the above types
* a `pointer` to one of the above types


#### Custom Marshaling


Is possible unmarshaling data and the key of a map by the `encoding.TextUnmarshaler` interface.

----

#### Example

```go
 //./main.go

package main

import (
	"fmt"

	"github.com/kataras/iris"
)

type Visitor struct {
	Username string
	Mail     string
	Data     []string `form:"mydata"`
}

func main() {

	iris.Get("/", func(ctx *iris.Context) {
		ctx.Render("form.html", nil)
	})

	iris.Post("/form_action", func(ctx *iris.Context) {
		visitor := Visitor{}
		err := ctx.ReadForm(&visitor)
		if err != nil {
			fmt.Println("Error when reading form: " + err.Error())
		}
		fmt.Printf("\n Visitor: %v", visitor)
	})

	fmt.Println("Server is running at :8080")
	iris.Listen(":8080")
}

```

```html

<!-- ./templates/form.html -->
<!DOCTYPE html>
<head>
<meta charset="utf-8">
</head>
<body>
<form action="/form_action" method="post">
<input type="text" name="Username" />
<br/>
<input type="text" name="Mail" /><br/>
<select multiple="multiple" name="mydata">
<option value='one'>One</option>
<option value='two'>Two</option>
<option value='three'>Three</option>
<option value='four'>Four</option>
</select>
<hr/>
<input type="submit" value="Send data" />

</form>
</body>
</html>

```


#### Example



##### In form html

- Use symbol `.` for access a field/key of a structure or map. (i.e, `struct.key`)
- Use `[int_here]` for access to index of a slice/array. (i.e, `struct.array[0]`)

```html
<form method="POST">
  <input type="text" name="Name" value="Sony"/>
  <input type="text" name="Location.Country" value="Japan"/>
  <input type="text" name="Location.City" value="Tokyo"/>
  <input type="text" name="Products[0].Name" value="Playstation 4"/>
  <input type="text" name="Products[0].Type" value="Video games"/>
  <input type="text" name="Products[1].Name" value="TV Bravia 32"/>
  <input type="text" name="Products[1].Type" value="TVs"/>
  <input type="text" name="Founders[0]" value="Masaru Ibuka"/>
  <input type="text" name="Founders[0]" value="Akio Morita"/>
  <input type="text" name="Employees" value="90000"/>
  <input type="text" name="public" value="true"/>
  <input type="url" name="website" value="http://www.sony.net"/>
  <input type="date" name="foundation" value="1946-05-07"/>
  <input type="text" name="Interface.ID" value="12"/>
  <input type="text" name="Interface.Name" value="Go Programming Language"/>
  <input type="submit"/>
</form>
```

##### Backend

You can use the tag `form` if the name of a input of form starts lowercase.

```go
package main

type InterfaceStruct struct {
    ID   int
    Name string
}

type Company struct {
  Public     bool      `form:"public"`
  Website    url.URL   `form:"website"`
  Foundation time.Time `form:"foundation"`
  Name       string
  Location   struct {
    Country  string
    City     string
  }
  Products   []struct {
    Name string
    Type string
  }
  Founders   []string
  Employees  int64
  
  Interface interface{}
}

func MyHandler(c *iris.Context) {
  m := Company{
      Interface: &InterfaceStruct{},
  }
  
  if err := c.ReadForm(&m); err != nil {
  		panic(err.Error())
  }
}
  
func main() {
  iris.Get("/bind_form", MyHandler)
  iris.Listen(":8080")
}
```
