# MySQL Client - Ecko Std Lib Package

A MySQL / MariaDB client for [Ecko](https://ecko.sh), written in Ecko
with no native client library. It speaks the classic protocol over `std.net`'s
raw sockets and implements `mysql_native_password` authentication (the SHA-1
scramble) using `std.hash`, `bytes`, and the bitwise operators.

## Install

```bash
ecko get github.com/ecko-lang/mysql-client --as mysql
```

`ecko get` vendors the package under
`./vendor/github.com/ecko-lang/mysql-client/` and pins a file-tree hash in
`ecko.sum`.

Without `--as`, `ecko get` records this dependency under the alias
`mysql_client` (the repo name with `-` mapped to `_`); `--as mysql` is what makes
`import mysql` work. It writes the entry as
`"mysql": "github.com/ecko-lang/mysql-client@0.39.0"`. Expand it to the map form in
your `ecko.json` to grant the network capability the client needs:

```json
{
  "dependencies": {
    "mysql": {
      "path": "github.com/ecko-lang/mysql-client",
      "version": "0.39.0",
      "grant": ["net"]
    }
  }
}
```

```ecko
import mysql
```

## Usage

```ecko
import mysql

db = mysql.connect({
    host: "127.0.0.1", port: 3306,
    user: "me", password: "secret", database: "app"
})                                      # or mysql.connect_tls({ ... })

rows = mysql.query(db, "select id, name from users order by id")
for r in rows {                         # rows are maps keyed by column name
    print("{r.id}: {r.name}")
}

# A sql { } block: every {hole} is a bound value.
name = "Ada"
mysql.query(db, sql { SELECT * FROM users WHERE name = {name} })

# Commits when the function returns; rolls back if it raises.
mysql.transaction(db, fn(tx) {
    mysql.query(tx, "UPDATE accounts SET balance = balance - ? WHERE id = ?", [100, 1])
    mysql.query(tx, "UPDATE accounts SET balance = balance + ? WHERE id = ?", [100, 2])
})

mysql.close(db)
```

## API

| function | notes |
|----------|-------|
| `connect({host, port, user, password, database})` | open a connection and authenticate (mysql_native_password) |
| `connect_tls({...})` | same, over TLS |
| `query(db, sql, params = [])` | run a statement, filling each `?` from `params`; returns a list of row maps (empty for non-SELECT) |
| `query(db, sql { ... })` | a `sql { }` block; its holes are bound the same way |
| `transaction(db, fn)` | run `fn(db)` in a transaction: commit on return, roll back and re-raise on error; returns what `fn` returned |
| `close(db)` | close the connection |

Values come back in text format: integers, dates, etc. arrive as strings, and
SQL `NULL` comes back as `null`. Column names are the map keys.

> **Scope.** Authentication is `mysql_native_password` (the MySQL 5.7 / MariaDB
> default, also available on MySQL 8 for users created with it). `caching_sha2_password`
> (the MySQL 8 default) and the server-side prepared protocol are planned.

**Pass values as `params`, never by building SQL.** Each `?` outside a quoted
string, a backtick identifier or a comment takes the next value:

```ecko fragment
mysql.query(db, "SELECT * FROM users WHERE email = ? AND active = ?", [email, true])
```

A string goes in as a hex literal with a charset introducer (`_utf8mb4 X'...'`),
so it contains no quote or backslash - nothing in the value can end the literal,
whatever the server's SQL mode or the connection's charset. Numbers, decimals,
booleans, `null` and bytes get their SQL literals; a list, map or a NaN is
refused with a `sql` error, and so is a count of `params` that does not match
the placeholders. With no `params` the SQL is sent exactly as written.

A `sql { }` block is the same `{ query, params }` value `std.sql` takes, and is
bound the same way. Pass it on its own: it carries its params, so a separate
list is refused.

### Transactions

`transaction(db, fn)` has the same shape as `std.sql`'s. It commits when `fn`
returns and gives back its value. If `fn` raises, it rolls back and re-raises
the same error. If the commit fails it rolls back and raises that error.

Transactions do not nest. MySQL itself would let them, by silently committing
the open transaction when a second one starts, so `transaction` checks first
and raises instead. That check costs one extra round trip per transaction.

## Errors

Every failure is `{ kind: "sql", driver: "mysql", message }`, the kind
`std.sql` raises, so one `catch` arm covers both. An error from the server also
carries `code`, its SQLSTATE, and `errno`, the MySQL error number (`1062` is a
duplicate key):

```ecko fragment
try {
    mysql.query(db, "INSERT INTO users (email) VALUES (?)", [email])
} catch (e) {
    match e.kind {
        "sql" => if get(e, "errno") == 1062 { print("already registered") } else { error(e) }
        _ => error(e)
    }
}
```

An error the server sends partway through a result set, such as a killed query,
is raised too, rather than read as a row.

**Migration from 0.38:** errors used to be `kind: "mysql"`. Match `"sql"`
instead, and check `e.driver` if you need to tell databases apart.

## Testing

```bash
ecko test          # offline: auth scramble, binding, blocks and transactions,
                   # partly against a scripted loopback server
ecko example.ecko  # live round trip against a MySQL/MariaDB server
```

## License

MIT
