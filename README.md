# pg_dump_sample

[![Build Status](https://travis-ci.org/dankeder/pg_dump_sample.svg?branch=master)](https://travis-ci.org/dankeder/pg_dump_sample)

This is a simple tool for dumping a sample of data from a PostgreSQL database.
The resulting dump can be loaded back into an new database using standard tools
(e.g. `psql(1)`).



## Why would I want this?

It is useful if you have a huge PostgreSQL database and you need a
database with a small dataset for testing or development.


## Features

- Data sampling: Dump either all rows of a table or only rows matching the
  specified query.
- Easy anonymization of data: You have full control over what's in the dump,
  it's very easy to create anonymized DB dumps.
- Templated queries: You can extract common parts of queries into one place.
  That's especially useful if you have e.g. a list of IDs the dump should contain.
- Table dependency awareness - Table A will be dumped before table B
  automatically if table B has foreign keys pointing to table A.
- Standard SQL - The produced dump files contain just a bunch of SQL commands.
  They can be loaded by standard tools such as `psql(1)` or processed further.


## How to build

The `pg_dump_sample` is written in Go. You need to [setup the Go compiler and
setup environment](https://golang.org/doc/install) first to build it.

    go get github.com/dankeder/pg_dump_sample

If it went well you should be able to run it:

    pg_dump_sample

If not check that you have `$GOPATH/bin` in your `$PATH`.


## How to use

A quick example:

    pg_dump_sample -f mydb.yaml -h mydbhost.dev -U postgres -o mydb_dump.sql mydb

Available command-line options:
         
    Usage:
      pg_dump_sample [options] database

    Application Options:
      -h, --host=          Database server host or socket directory (default: local socket) [$PGHOST]
      -p, --port=          Database server port (default: 5432) [$PGPORT]
      -U, --username=      Database user name (default: current user) [$PGUSER]
      -w, --no-password    Don't prompt for password
      -f, --manifest-file= Path to manifest file
      -o, --output-file=   Path to the output file
      -s, --tls            Use SSL/TLS database connection
          --help           Show help

The available command-line options are heavily inspired by
[`pg_dump(1)`](http://www.postgresql.org/docs/9.4/static/app-pgdump.html).
Anyone familiar with it should feel right at home.

It is also possible to use environmental variables to set the options.
Note that the environmental variables have lower precenence than command-line
options. The supported variables are:

| Environmental variable    | Description                         |
|  ------------------------ | ----------------------------------- |
| `PGHOST`                  | `-h, --host`                        |
| `PGPORT`                  | `-p, --port`                        |
| `PGUSER`                  | `-U, --username`                    |
| `PGPASSWORD`              | Used to set the password. Use of this environment variable is not recommended for security reasons (some operating systems allow non-root users to see process environment variables via ps)
| `PGDATABASE`              | database                            |


### Manifest file

The main difference between `pg_dump_sample` and `pg_dump(1)` is that
`pg_dump_sample` requires a manifest file describing how to dump the database.
The manifest file is a YAML file describing what tables to dump and how to dump
them.

A quick example:

    ---
    vars:
      # Condition to dump only certain users
      matching_user_id: "(users.id BETWEEN 1000 AND 2000)"

    tables:
      # Dump everything from table "consts"
      - table: consts

      # Dump only matching users
      - table: users
        query: "SELECT * FROM users WHERE {{matching_user_id}}"
        post_actions:
          - "SELECT pg_catalog.setval('users_id_seq', MAX(id) + 1, true) FROM users"

      # Dump only tickets that were bought by matching users
      - table: tickets
        query: >
          SELECT purchases.* FROM purchases, users
          WHERE
            purchases.buyer_id = users.id
            AND {{matching_user_id}}


Currently these top-level keys are available:

#### `vars`

Definitions of variables which will be used to replace placeholders in queries.

#### `tables`

List of tables to dump. Tables are dumped in the order they are specified in the
manifest file, with one exception: if the table contains foreign keys
referencing another table, the referenced table will be dumped first. This is to
ensure that the dump can be loaded later without errors.

By default all rows of the table will be dumped. If you don't want to dump all
the rows use the `query` to specify a SELECT SQL statement which returns the
rows you want to dump.


## TODO

- Use separate vars files to override vars from manifest?
- Allow setting vars using command-line options?


## Contributing

- If you find a new bug, a missing feature, etc. please create a ticket in Isuses.
- Contributions are very welcome - just open a pull-request.


# Licence

MIT


# Author

Dan Keder <dan.keder@gmail.com>
