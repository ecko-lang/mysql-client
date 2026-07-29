# mysql-client

## `native_password(password, scramble)`

mysql_native_password: SHA1(pw) XOR SHA1(scramble + SHA1(SHA1(pw))).

## `connect(cfg)`

connect({ host, port, user, password, database }) -> a connection handle.

## `connect_tls(cfg)`

Connect to MySQL over TLS and complete authentication. Same `cfg` as
`connect`. Needs the `net` capability.

## `close(sock)`

Close the connection.

## `query(sock, sql)`

query(sock, sql) -> a list of row maps (empty for non-SELECT statements).
