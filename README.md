r2d2
====

A generic connection pool for Rust.

[![Build Status](https://travis-ci.org/sfackler/r2d2.svg?branch=master)](https://travis-ci.org/sfackler/r2d2) [![Latest Version](https://img.shields.io/crates/v/r2d2.svg)](https://crates.io/crates/r2d2)

[Documentation](https://sfackler.github.io/r2d2/doc/v0.6.1/r2d2)

Opening a new database connection every time one is needed is both inefficient
and can lead to resource exhaustion under high traffic conditions. A connection
pool maintains a set of open connections to a database, handing them out for
repeated use.

r2d2 is agnostic to the connection type it is managing. Implementors of the
`ManageConnection` trait provide the database-specific logic to create and
check the health of connections.

A (possibly not exhaustive) list of adaptors for different databases:

Database | Database Crate                                                 | Adaptor Crate
-------- | -------------------------------------------------------------- | -------------
Postgres | [postgres](https://github.com/sfackler/rust-postgres)          | [r2d2-postgres](https://github.com/sfackler/r2d2-postgres)
Redis    | [redis](https://github.com/mitsuhiko/redis-rs)                 | [r2d2-redis](https://github.com/nevdelap/r2d2-redis)
MySQL    | [mysql](https://github.com/blackbeam/rust-mysql-simple)        | [r2d2-mysql](https://github.com/outersky/r2d2-mysql)
SQLite   | [rusqlite](https://github.com/jgallagher/rusqlite)             | [r2d2-sqlite](https://github.com/ivanceras/r2d2-sqlite)
Neo4j    | [rusted-cypher](https://github.com/livioribeiro/rusted-cypher) | [r2d2-cypher](https://github.com/flosse/r2d2-cypher)

# Example

Using an imaginary "foodb" database.
```rust
use std::thread;

extern crate r2d2;
extern crate r2d2_foodb;

fn main() {
    let config = r2d2::Config::builder()
        .error_handler(Box::new(r2d2::LoggingErrorHandler))
        .build();
    let manager = r2d2_foodb::FooConnectionManager::new("localhost:1234");

    let pool = r2d2::Pool::new(config, manager).unwrap();

    for _ in 0..20i32 {
        let pool = pool.clone();
        thread::spawn(move || {
            let conn = pool.get().unwrap();
            // use the connection
            // it will be returned to the pool when it falls out of scope.
        })
    }
}
```
