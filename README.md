# MacOS PostgreSQL Guide

This guide is written with PostgreSQL 10.1, Homebrew 1.3.9, and MacOS 10.13.2.

## Installing with Homebrew

To install PostgreSQL on MacOS with homebrew, run the following commands in terminal:

```bash
# Make sure brew is healthy and working with the latest formulae
brew update
brew doctor

# Install PostgreSQL
brew install postgres
```

## Notes Regarding Homebrew

Homebrew will install PostgreSQL in it's own directory in `/usr/local/Cellar/` and symlink it's binaries in `/usr/local/bin/`

Some configuration files, such as `pg_hba.conf`, can be found in `/usr/local/var/postgres/`

Most tutorials will show that the user `postgres` is the default user. However, when installing PostgreSQL with brew, the default user and db superuser will be the user that ran the installation.

## Starting, Stopping, and Restarting PostgreSQL

Use the following commands to start, stop, and restart postgres

```bash
# Start the server
brew services start postgres

# Stop the server
brew services stop postgres

# Restart the server
brew services restart postgres
```

### Run PostgreSQL on Boot with `launchctl`

When installed with brew, postgres will come with a `.plist` file that can be used to run it on boot with `launchctl`.

The file path should be `/usr/local/Cellar/postgresql/<postgres version>/homebrew.mxcl.postgresql.plist`.

For example, if your postgres version is 10.1, the `.plist` file will be at the following location.
```bash
/usr/local/Cellar/postgresql/10.1/homebrew.mxcl.postgresql.plist
```

To run postgres on boot, first copy the `.plist` file to your `~/Library/LaunchAgents/` folder (you may need to create the folder if it doesn't already exist).

```bash
cp /usr/local/Cellar/postgresql/10.1/homebrew.mxcl.postgresql.plist ~/Library/LaunchAgents/
```

Then, add the `.plist` file to `launchctl` like so:

```bash
launchctl load -w homebrew.mxcl.postgresql.plist
```

PostgreSQL will now load on system start.

To disable postgres from loading on start, use the `launchctl unload` command:

```bash
launchctl unload -w homebrew.mxcl.postgresql.plist
```
## Basic Security

[Securing PostgreSQL on Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-secure-postgresql-on-an-ubuntu-vps)

## Create a Database

```sql
CREATE DATABASE <db_name> OWNER <role_name>;
```

When creating a new database, the new db will be created from a template. All databases, including templates, can be viewed with the `\l` command from the psql client.

[PostgreSQL CREATE DATABASE](https://www.postgresql.org/docs/10/static/sql-createdatabase.html)

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
CREATE ROLE test WITH LOGIN ENCRYPTED PASSWORD 'thePassword';
```

When specifying permissions when creating roles, they do not need to be separated with commas. Here, `LOGIN` gives the user login privileges and `ENCRYPTED PASSWORD` instructs the server to encrypt the stored password.

This query can also be written as `CREATE USER test WITH ENCRYPTED PASSWORD 'thePassword'`.

[Roles and Grant Permissions in PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)

## Granting Permissions

Grant permissions to all tables in a schema:
```sql
GRANT <permissions> ON ALL TABLES IN SCHEMA <schema> TO <role>;
```

[PostgreSQL GRANT Documentation](https://www.postgresql.org/docs/10/static/sql-grant.html)

[StackOverflow Post on GRANT](https://stackoverflow.com/a/17355059)

## Creating Schema

Schema can be used to namespace tables so as not to create naming collisions.

[PostgreSQL Schema Documentation](https://www.postgresql.org/docs/current/static/ddl-schemas.html)

http://www.postgresqlforbeginners.com/2010/12/schema.html

## Inheriting permissions

[PostgreSQL Role Inheritance Documentation](https://www.postgresql.org/docs/10/static/role-membership.html)

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

# Resources
