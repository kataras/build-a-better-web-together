# Send e-mails
Sending plain or rich content e-mails is an easy process with Iris.

Configuration 

```go
// Mail keeps the configs for mail sender service
type Mail struct {
	// Host is the server mail host, IP or address
	Host string
	// Port is the listening port
	Port int
	// Username is the auth username@domain.com for the sender
	Username string
	// Password is the auth password for the sender
	Password string
	// UseCommand enable it if you want to send e-mail with the mail command  instead of smtp
	//
	// Host,Port & Password will be ignored
	// ONLY FOR UNIX
	UseCommand bool
}

```

**Example**

> Note that this is not working (panic: EOF) until you define your own  host, port, username, password and recipients.

File: ` ./main.go ` 
```go
package main

import (
	"github.com/kataras/iris"
)

func init() {
	configuration := iris.Config()
	configuration.Mail.Host = "smtp.gmail.com"
	configuration.Mail.Port = 465
	configuration.Mail.Username = "YourUsername@gmail.com"
	configuration.Mail.Password = "yourP@ssw0rd$1!123"
}

func main() {
	// define the recipients
	to := []string{"receiver@gmail.com", "recipient@hotmail.com"}
	subject := "Iris mail test"

	// 1.
	// send a plain text message
	messageBody := "Hello from Iris, this is a plain e-mail."
	err := iris.Mail().Send(to, subject, messageBody)
	if err != nil {
		panic(err)
	}

	// 2.
	// send html template as message ( from ./templates/mail_body.html)
	messageBody, _ = iris.Templates().RenderString("mail_body.html", iris.Map{"Message": "<b> Hello from Iris</b> this is a <h1 style='color:blue'> Rich e-mail!</h1>"})
	err = iris.Mail().Send(to, subject, messageBody)
	if err != nil {
		panic(err)
	}

	iris.Listen(":8080")
}

```

File: `./templates/mail_body.html` 

```html
<html>
<head></head>
<body>

	<h1> From iris server:</h1>
	<p>{{.Message}}</p>

</body>
</html>
```
