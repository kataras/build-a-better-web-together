# Body binder

Body binder reads values from the body and set them to a specific object.


## Forms
The form binding came from a fast third party package named [formam](https://github.com/monoculum/formam).

### Types

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
* `custom types` to one of the above types
* a `pointer` to one of the above types

**NOTE**: the nesting in `maps`, `structs` and `slices` can be [ad infinitum](https://en.wikipedia.org/wiki/Ad_infinitum).

Custom Marshaling
-----------------

Is possible unmarshaling data and the key of a map by the `encoding.TextUnmarshaler` interface.

Usage
-----

### In form html

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

### In golang

You can use the tag `formam` if the name of a input of form starts lowercase.

```go
type InterfaceStruct struct {
    ID   int
    Name string
}

type Company struct {
  Public     bool      `formam:"public"`
  Website    url.URL   `formam:"website"`
  Foundation time.Time `formam:"foundation"`
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

iris.Get("/bind_form", MyHandler)

iris.Listen(":8080")
```
