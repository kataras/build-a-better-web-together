# Model Validation

Iris does not contain built-in method for validating request data such as "Models". However, you are not limited. In this example we will learn how to use the [**go-playground/validator.v9**](https://github.com/go-playground/validator) for request values validation.

Check the full docs on tags usage [here](http://godoc.org/gopkg.in/go-playground/validator.v9#hdr-Baked_In_Validators_and_Tags).

```bash
$ go get gopkg.in/go-playground/validator.v9
```

Note that you need to set the corresponding binding tag on all fields you want to bind. For example, when binding from JSON, set `json:"fieldname"`.

```go
package main

import (
    "fmt"

    "github.com/kataras/iris"

    "gopkg.in/go-playground/validator.v9"
)

// User contains user information.
type User struct {
    FirstName      string     `json:"fname"`
    LastName       string     `json:"lname"`
    Age            uint8      `json:"age" validate:"gte=0,lte=130"`
    Email          string     `json:"email" validate:"required,email"`
    FavouriteColor string     `json:"favColor" validate:"hexcolor|rgb|rgba"`
    Addresses      []*Address `json:"addresses" validate:"required,dive,required"`
}

// Address houses a users address information.
type Address struct {
    Street string `json:"street" validate:"required"`
    City   string `json:"city" validate:"required"`
    Planet string `json:"planet" validate:"required"`
    Phone  string `json:"phone" validate:"required"`
}

// Use a single instance of Validate, it caches struct info.
var validate *validator.Validate

func main() {
    validate = validator.New()

    // Register validation for 'User'
    // NOTE: only have to register a non-pointer type for 'User', validator
    // internally dereferences during it's type checks.
    validate.RegisterStructValidation(UserStructLevelValidation, User{})

    app := iris.New()
    app.Post("/user", func(ctx iris.Context) {
        var user User
        if err := ctx.ReadJSON(&user); err != nil {
            // [handle error...]
        }

        // Returns InvalidValidationError for bad validation input,
        // nil or ValidationErrors ( []FieldError )
        err := validate.Struct(user)
        if err != nil {

            // This check is only needed when your code could produce
            // an invalid value for validation such as interface with nil
            // value most including myself do not usually have code like this.
            if _, ok := err.(*validator.InvalidValidationError); ok {
                ctx.StatusCode(iris.StatusInternalServerError)
                ctx.WriteString(err.Error())
                return
            }

            ctx.StatusCode(iris.StatusBadRequest)
            for _, err := range err.(validator.ValidationErrors) {
                fmt.Println()
                fmt.Println(err.Namespace())
                fmt.Println(err.Field())
                fmt.Println(err.StructNamespace())
                fmt.Println(err.StructField())
                fmt.Println(err.Tag())
                fmt.Println(err.ActualTag())
                fmt.Println(err.Kind())
                fmt.Println(err.Type())
                fmt.Println(err.Value())
                fmt.Println(err.Param())
                fmt.Println()
            }

            return
        }

        // [save user to database...]
    })

    app.Run(iris.Addr(":8080"))
}

func UserStructLevelValidation(sl validator.StructLevel) {
    user := sl.Current().Interface().(User)

    if len(user.FirstName) == 0 && len(user.LastName) == 0 {
        sl.ReportError(user.FirstName, "FirstName", "fname", "fnameorlname", "")
        sl.ReportError(user.LastName, "LastName", "lname", "fnameorlname", "")
    }
}
```

Example request of JSON form:

```javascript
{
    "fname": "",
    "lname": "",
    "age": 45,
    "email": "mail@example.com",
    "favColor": "#000",
    "addresses": [{
        "street": "Eavesdown Docks",
        "planet": "Persphone",
        "phone": "none",
        "city": "Unknown"
    }]
}
```

Example can be found at: [https://github.com/kataras/iris/tree/master/_examples/http_request/read-json-struct-validation/main.go](https://github.com/kataras/iris/tree/master/_examples/http_request/read-json-struct-validation/main.go).

