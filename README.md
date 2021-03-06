
# Guide to PostgreSQL on MacOS 2019

This guide is written with PostgreSQL 11.4, Homebrew 2.1.9, and MacOS 10.14.5

---

# Installation

```bash
$ brew update
$ brew doctor

$ brew install postgresql
```

This should leave you with a Postgres db at /usr/local/var/postgres.

If the initial database was not created on install, you need to initialize a database cluster:

```bash
$ initdb /usr/local/var/postgres
```

## Create/Drop a Database

[Make sure the server is running](#managing-the-postgresql-service), then use the `createdb` command to create a new database:

```bash
# create a new database called test_db
$ createdb test_db

# drop a database
$ dropdb test_db
```

## Updating Postgres

```bash
# Fetch newest version of homebrew and update all formulae
$ brew update

# Upgrade PostgreSQL itself
$ brew upgrade postgresql
```

## Notes Regarding Homebrew

Homebrew will install PostgreSQL in it's own directory in `/usr/local/Cellar/` and symlink it's binaries in `/usr/local/bin/`

Some configuration files, such as `pg_hba.conf`, can be found in `/usr/local/var/postgres/`

Most tutorials will show that the user `postgres` is the default user. However, when installing with homebrew, the default user and db superuser will be the user that ran the installation.

---

# Managing the PostgreSQL Service

## Start/Stop/Restart

```bash
# Start the service
$ brew services start postgresql

# Stop the service
$ brew services stop postgresql

# Restart the service
$ brew services restart postgresql
```

If you have created a different database cluster other than the default, or the default cluster is not running/managed by homebrew you can use `pg_ctl` to start and stop it:

```bash
# Start the server
$ pg_ctl -D /path/to/db/ start

# Stop the server
$ pg_ctl -D /path/to/db/ stop
```

## Run Postgres on Boot with `launchctl`

When installed with brew, postgres will come with a `.plist` file that can be used to run it on boot with `launchctl`.

The file path should be `/usr/local/Cellar/postgresql/<postgres version>/homebrew.mxcl.postgresql.plist`.

For example, if your postgres version is 11.4, the `.plist` file will be at `/usr/local/Cellar/postgresql/11.4/homebrew.mxcl.postgresql.plist`

To run postgres on boot, first copy the `.plist` file to your `~/Library/LaunchAgents/` folder (you may need to create the folder if it doesn't already exist). Then load it with `launchctl`:

```bash
# Copy the .plist file to LaunchAgents
cp /usr/local/Cellar/postgresql/11.4/homebrew.mxcl.postgresql.plist ~/Library/LaunchAgents/

# Add to launchctl
launchctl load -w homebrew.mxcl.postgresql.plist
```
PostgreSQL will now load on system start.

To disable postgres from loading on start, use the `launchctl unload` command:

```bash
launchctl unload -w homebrew.mxcl.postgresql.plist
```

---

# Connect to the Database Server

To connect to the server once it is running use the `psql` command:

```bash
# Connect as the current shell user
$ psql

# Connect as the 'unicorn' user (should prompt for password)
$ psql -U unicorn

# Connect as the 'unicorn' user, force password prompt
$ psql -U unicorn -W
```

---

# Basic Security

[Securing PostgreSQL on Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

---

# Managing Permissions

In postgres, a database must have an owner. So, before we start looking at database, schema, and table creation, we have to learn how to manage roles.

## Important Note

It is important to note that, in postgres, unquoted identifiers are always folded to lower case. In other words, any identifier (table name, role, schema, etc) that is not enclosed in double quotes `"` `"`, will be folded, or converted, to it's lower case form.

For example, in the query `CREATE ROLE myRole WITH LOGIN` the identifier `myRole` will be converted to `myrole` when the role is created, unless you wrap it in double quotes. In order to create a case sensitive role `myRole`, you would have to run to query like this:

```sql
CREATE ROLE "myRole" WITH LOGIN;
```

It is also important to note that values (string literals), are to be enclosed in single quotes `'` `'`, while double quotes are reserved for identifiers.

As a general rule, it is best to use lower case names for identifiers in order to avoid having to wrap the identifier in double quotes every time you want to use it for the lifetime of the object. If readability is an issue, consider using snake_case. This also has the benefit of being easily distinguishable from the UPPER CASE convention for SQL key words.

For the sake of clarity and readability, all SQL key words will be UPPER CASE while all identifiers will be snake_case.

[PostgreSQL Identifiers and Keywords](https://www.postgresql.org/docs/current/static/sql-syntax-lexical.html#SQL-SYNTAX-IDENTIFIERS)

## Creating Roles

Postgres doesn't make much of a distinction between a user and a role. A user is simply a role that can log in.

The syntax for creating a role is as follows:

```sql
CREATE ROLE <role_name> WITH <optional_permissions>
```

There is also a `CREATE USER` syntax; however, the only difference between `CREATE ROLE` and `CREATE USER` is that the latter will automatically grant `LOGIN` permissions, while the former will not.

[This document](https://www.postgresql.org/docs/current/static/sql-createrole.html) outlines all of the available `optional_permissions` that can be specified.

The most common optional permissions are:
* LOGIN | NOLOGIN
* SUPERUSER | NOSUPERUSER
* CREATEDB | NOCREATEDB
* CREATEROLE | NOCREATEROLE

To create a role named test with encrypted password, the command would be as follows:

```sql
CREATE ROLE my_role WITH LOGIN ENCRYPTED PASSWORD 'thePassword';
```

When specifying permissions when creating roles, they do not need to be separated with commas. Here, `LOGIN` gives the user login privileges and `ENCRYPTED PASSWORD` instructs the server to encrypt the stored password.

This query can also be written as `CREATE USER test WITH ENCRYPTED PASSWORD 'thePassword'`.

[Roles and Grant Permissions in PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)

More advanced role and permission management will follow later, let's create some tables.


---


# Setting Up a Database, Schema, and Table

## Create a Database

```sql
CREATE DATABASE <db_name> OWNER <role_name>;
```

When creating a new database, the new db will be created from a template. All databases, including templates, can be viewed with the `\l` command from the psql client.

[PostgreSQL CREATE DATABASE](https://www.postgresql.org/docs/current/static/sql-createdatabase.html)

## Creating Schema

Schema can be used to namespace tables so as not to create naming collisions.

[PostgreSQL Schema Documentation](https://www.postgresql.org/docs/current/static/ddl-schemas.html)

http://www.postgresqlforbeginners.com/2010/12/schema.html

## Creating Tables

[PostgreSQL CREATE TABLE](https://www.postgresql.org/docs/current/static/sql-createtable.html)

[PostgreSQL Data Types](https://www.postgresql.org/docs/current/static/datatype.html)


---

# More Permissions management

## Granting Permissions

Some examples using the role myRole.

By default, each database has a first schema called `public`. When creating a new table in a database, if another schema is not specified, the new table will be part of the `public` schema. All users have access to `public` schema by default. So, one of the first things we want to do is create a new schema.

```sql
CREATE SCHEMA my_schema;
```

The creator of a schema by default has access to use it. But oftentimes we want to grant permission to other roles or groups. Even if a role or group has permission to `SELECT` or perform other table functions, if they do not have access to use the schema a table belongs too, then they cannot use it.

Grant usage of schema my_schema to user my_user:
```sql
GRANT USAGE ON SCHEMA my_schema TO my_user;
```

Once a role/group has access to use the schema, we can start granting permissions. *Note that we can do this in any order, as long as usage has been granted to a schema and permissions granted to a table in that schema when the user tries to use it, it makes no difference in what order they were added.*

Grant permissions to all tables in a schema:
```sql
GRANT SELECT, INSERT, UPDATE, DELETE ON ALL TABLES IN SCHEMA my_schema TO my_user;
```

Grant all permission on my_table in the my_schema schema:
```sql
GRANT ALL PERMISSIONS ON my_schema.my_table TO my_user;
```

Permissions can be specified on an entire table as above, or on specific columns:
```sql
GRANT SELECT (id), INSERT (id) ON my_schema.my_table TO my_user;
```

[PostgreSQL GRANT Documentation](https://www.postgresql.org/docs/current/static/sql-grant.html)

[StackOverflow Post on GRANT](https://stackoverflow.com/a/17355059)

## Inheriting permissions

[PostgreSQL Role Inheritance Documentation](https://www.postgresql.org/docs/current/static/role-membership.html)

The `public` schema has a default `GRANT` of all rights to the role `public`, which every user/group is a member of. To allow access to a schema you created, you must `GRANT USAGE` on the schema to the role that needs to access it.

```sql
CREATE SCHEMA foo;

CREATE TABLE foo.t(x int);

CREATE ROLE foomaster NOSUPERUSER NOLOGIN INHERIT;

CREATE ROLE fooslave PASSWORD 'thePassword' LOGIN INHERIT IN ROLE foomaster;

GRANT USAGE ON SCHEMA foo TO foomaster;

SET ROLE fooslave;

SELECT * FROM foo.t; # ERROR: permission denied for relation t

SET ROLE postgres;

GRANT SELECT ON foo.t TO foomaster;

SET ROLE fooslave;

SELECT * FROM foo.t; # No error ;)
```

[StackOverflow Post](https://dba.stackexchange.com/a/135545)

## Sequences

[Blog Post on Sequences](http://www.neilconway.org/docs/sequences/)
