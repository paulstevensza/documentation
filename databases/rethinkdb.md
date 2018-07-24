# RethinkDB

I love RethinkDB. Database and JSON API all in one? Sign me up! Support for websockets and realtime connections? Awesome! I
was gutted when RethinkDB the company closed their doors, but encouraged when it was announced that the Linux Foundation
was taking over the project. After letting it lie for a while, I went back to the project today and was delighted to see
that development is still active, and a running instance of RethinkDB was a quick `docker run` away.

## Documentation

* [RethinkDB documentation site](https://rethinkdb.com/docs/)
* [RethinkDB @ GitHUb](https://github.com/rethinkdb/rethinkdb)

## Running in Docker

Bootstrap an instance of RethinkDB in Docker:

```
$ docker run -d -P --name rethink1 rethinkdb
```

On my machine, it maps the default admin UI running on `8080/tcp` to `32770/tcp`, so browse to
[https://0.0.0.0:32770](https://0.0.0.0:32770) to access the UI.
