Working with Databases
======================

## Table of Contents

[...]

### Listing Long-Running Queries

To launch the database console, you can directly use the `psql` command or
use the built-in wrapper command `bin/rails dbconsole -p`. This command will
supply all required host parameters, including the password (when `-p` is
provided). See `bin/rails dbconsole --help` for more details.

Our staging/production systems offer the shell alias `rails-dbconsole` to
execute the command directly after connecting to the host. Note that by
default, you will connect to the main database. If you wish to operate on the
`logs` database, you need to append the `--db logs` parameter.

While connected to the database, you can list running processes by querying
built-in tables using SQL.

```sql
SELECT
    pid,
    usename,
    query_start,
    now() - query_start AS duration,
    LEFT(query, 100) AS query_snippet
FROM
    pg_stat_activity
WHERE
    state = 'active'
    AND now() - query_start > interval '60 seconds'
ORDER BY
    duration DESC;
```

This query will list all processes that have been running for more than 1
minute (adjust the interval to your needs). It will also only show the first
100 characters of the query for easier display. You can either modify the trim
level or remove it entirely.

Note: You can switch to "column view" using the `\x` command.

### Stopping Queries

To stop a running query, you need to find the PID of the query and then
use `SELECT pg_cancel_backend(PID);`. This command will cancel the
selected query. In rare cases, this might not be sufficient, and `SELECT
pg_terminate_backend(PID);` might be required. The latter will terminate the
whole connection used to execute this query.

![Screenshot](assets/psql-demo-databases.png)
