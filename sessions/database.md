# Sessions Database

Sometimes you need a backend storage, i.e redis, which will keep your session data on server restarts and scale horizontally.

Registering a session database can be done through a single call of `sessions.UseDatabase(database)`.

Iris implements three (3) builtin session databases for [redis](https://redis.io/), [badger](https://github.com/dgraph-io/badger) and [boltdb](github.com/etcd-io/bbolt). These session databases are implemented via the [iris/sessions/sessiondb](https://github.com/kataras/iris/tree/master/sessions/sessiondb) subpackage which you'll have to import.

1. import the `gitbub.com/kataras/iris/sessions/sessiondb/redis or boltdb or badger`,
2. initialize the database and
3. register it

For example, to register the redis session database:

```go
import (
    "github.com/kataras/iris"
    "github.com/kataras/iris/sessions"

    // 1. Import the session database.
    "github.com/kataras/iris/sessions/sessiondb/redis"
)

// 2. Initialize the database.
// Replace with your running redis' server settings:
db := redis.New(redis.Config{
    Network:   "tcp",
    Addr:      "127.0.0.1:6379",
    Timeout:   time.Duration(30) * time.Second,
    MaxActive: 10,
    Password:  "",
    Database:  "",
    Prefix:    "",
})

// Close connection when control+C/cmd+C
iris.RegisterOnInterrupt(func() {
    db.Close()
})

sess := sessions.New(sessions.Config{
    Cookie:       "sessionscookieid",
    AllowReclaim: true,
})

// 3. Register it.
sess.UseDatabase(db)

// [...]
```

**boltdb**

```go
import "os"
import "github.com/kataras/iris/sessions/sessiondb/boltdb"

db, err := boltdb.New("./sessions.db", os.FileMode(0750))
```

**badger**

```go
import "github.com/kataras/iris/sessions/sessiondb/badger"

db, err := badger.New("./data")
```
