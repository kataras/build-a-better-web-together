# Using Handler API
HandlerAPI is any custom struct which has an `*iris.Context` field.

Instead of writing Handlers/HandlerFuncs for eachone API routes, you can use the ` iris.API` function.

**For example**, for a user API you need some of these routes:

- GET `/users` , for selecting all
- GET`/users/:id` , for selecting specific
- PUT `/users` , for inserting
- POST `/users/:id` , for updating
- DELETE `/users/:id` , for deleting






