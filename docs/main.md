# mysql-client

## `connect(cfg)`

connect({ host, port, user, password, database }) -> a connection handle.

## `connect_tls(cfg)`

Connect to MySQL over TLS and complete authentication. Same `cfg` as
`connect`. Needs the `net` capability.

## `close(sock)`

Close the connection.

## `query(sock, sql, params = [])`

query(sock, sql, params = []) -> a list of row maps (empty for non-SELECT
statements). Each `?` in `sql` takes the next value of `params`, written as a
SQL literal that nothing in the value can break out of - pass user input
here rather than building it into the SQL string. A `sql { }` block works in
place of the string and the list.

## `transaction(sock, body)`

transaction(db, fn) runs `fn(db)` inside a transaction: it commits when `fn`
returns, and rolls back and re-raises if `fn` raises. A commit that fails is
rolled back too, and raises. Returns what `fn` returned. Transactions do not
nest: starting one inside another raises.
