# Migrations & Database Tasks

This document shares some tips on dealing with database in development.

## Initial setup

Run:

```sh
bin/setup
```

To create and provision the database for the first time.

## Running pending migrations

When facing `PendingMigration` error (or similar), run the following command:

```sh
bundle exec rails db:migrate
```

Or alternatively:

```sh
bin/update
```

**NOTE:** `test` schema is maintained automatically when tests are running.

## Resetting database state

Run the following command to re-create the database and seed it with fresh data:

```sh
bundle exec rails db:reset
```

You can also "clean" (truncate) all the tables by running:

```sh
bundle exec rails db:truncate_all

# or if you want to re-seed as well
bundle exec rails db:seed:replant
```
