# mysql

A MySQL / MariaDB client for [Ecko](https://ecko.sh), written in pure Ecko
with no native client library. It speaks the classic protocol over `std.net`'s
raw sockets and implements `mysql_native_password` authentication (the SHA-1
scramble) using `std.hash`, `bytes`, and the bitwise operators.

## Install

```bash
ecko add https://github.com/ecko-sh/mysql-client   # or a local path / release zip
```

`ecko add` vendors the package into `./vendor/mysql/` and pins it in
`ecko.lock`. Grant it the network capability in your `ecko.json`:

```json
{
  "dependencies": {
    "mysql": { "source": "https://github.com/ecko-sh/mysql-client", "grant": ["net"] }
  }
}
```

## Use

```ecko
import mysql

db = mysql.connect({
    host: "127.0.0.1", port: 3306,
    user: "me", password: "secret", database: "app"
})                                      # or mysql.connect_tls({ ... })

rows = mysql.query(db, "select id, name from users order by id")
for r in rows {                         # rows are maps keyed by column name
    print(get(r, "id") + ": " + get(r, "name"))
}

mysql.close(db)
```

## API

| function | notes |
|----------|-------|
| `connect({host, port, user, password, database})` | open a connection and authenticate (mysql_native_password) |
| `connect_tls({...})` | same, over TLS |
| `query(db, sql)` | run a statement; returns a list of row maps (empty for non-SELECT) |
| `close(db)` | close the connection |
| `native_password(password, scramble)` | the auth scramble on its own, used by the offline test |

Values come back in text format: integers, dates, etc. arrive as strings, and
SQL `NULL` comes back as `null`. Column names are the map keys.

> **Scope.** Authentication is `mysql_native_password` (the MySQL 5.7 / MariaDB
> default, also available on MySQL 8 for users created with it). `caching_sha2_password`
> (the MySQL 8 default) and the extended/prepared protocol are planned. Statements
> are sent as text (simple protocol), so quote and escape untrusted input yourself.

## Testing

```bash
ecko test          # offline: checks the auth scramble against a known vector, no server needed
ecko example.ecko  # live round trip against a MySQL/MariaDB server
```

## License

MIT
