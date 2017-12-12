# MacOS PostgreSQL Guide

This guide is written with PostgreSQL 10.1 in mind.

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

## Creating Roles

Postgres doesn't make much of a distinction between a user and a role. A user is simply a role that can log in.

The syntax for creating a role is as follows:

```sql
CREATE ROLE <role_name> WITH <optional_permissions>
```

There is also a `CREATE USER` syntax; however, the only difference between `CREATE ROLE` and `CREATE USER` is that the latter will automatically grant `LOGIN` permissions, while the former will not.

[This document](https://www.postgresql.org/docs/current/static/sql-createrole.html) outlines all of the available `optional_permissions` that can be specified.

# Resources
[Roles and Grant Permissions in PostgreSQL](https://www.digitalocean.com/community/tutorials/how-to-use-roles-and-manage-grant-permissions-in-postgresql-on-a-vps--2)
