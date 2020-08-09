# Binary

**Content-Type: "application/octet-stream"**

A MIME attachment with the content type `"application/octet-stream"` is a **binary** file. Typically, it will be an application or a document that must be opened in an application, such as a spreadsheet or word processor. If the attachment has a filename extension associated with it, you may be able to tell what kind of file it is. A .exe extension, for example, indicates it is a Windows or DOS program (executable), while a file ending in .doc is probably meant to be opened in Microsoft Word.

No matter what kind of file it is, an application/octet-stream attachment **is rarely viewable** in web client. After downloading an attachment users must then open the attachment in the appropriate application to view its contents.

The `Context.Binary(data)` is the method which sends binary responses to the client. It accepts `[]byte` value.

```go
func handler(ctx iris.Context) {
    ctx.Binary([]byte("Some binary data here."))
}
```

You may also want to see the following methods for serving files:

> Reference: [Context-File-Server](../file-server/context-file-server.md)

<!-- slide:break-100 -->
