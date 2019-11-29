# PostgreSQL

**Production version: 11.2.**

We use Heroku provided PostgreSQL.

## Limits vs. Concurrency

One of the common issues that could cause downtimes is the lack of DB connections.

We should keep in mind that there is a limit on the number of connections in PostgreSQL:

```sh
# check the current limit using Heroku CLI
$ heroku pg:info
# => ...
# => Connections:           X/Y
```

Each Rails process maintains a fixed-size pool of DB connections (configured via `RAILS_MAX_THREADS` env variable and defaults to 5).

For `web` dynos the number of connections is equal to `WEB_CONCURRENCY * RAILS_MAX_THREADS`.

**NOTE:** when [Preboot](https://devcenter.heroku.com/articles/preboot) is enabled for Heroku app (enabled for production), the number of `web` dynos at a time could be doubled (during the _preboot_ phase). Thus we should add a multiplier (2) to the final formula.

For `worker` dynos we set `RAILS_MAX_THREADS` equal to `SIDEKIQ_CONCURRENCY` (the number of Sidekiq threads) thus making the DB pool size equal to `SIDEKIQ_CONCURRENCY` as well.

For `scheduler` dynos we set `RAILS_MAX_THREADS` to 1–only one DB connection.

We also need connections for one-off dynos (the same number as for the `web` dynos) and for `psql` sessions (so we need to keep some free connections for that, say 10).

The final formula for the total number of connections is the following:

```
(web_dyno_count * WEB_CONCURRENCY * RAILS_MAX_THREADS) * 2 (preboot) +
(worker_dyno_count * SIDEKIQ_CONCURRENCY) +
(scheduler_dyno_count) +
10
```

For example, with two web dynos with `WEB_CONCURRENCY=2`, `RAILS_MAX_THREADS=5`, one worker with `SIDEKIQ_CONCURRENCY=10` and one scheduler we have:

```sh
2 * 2 * 5 * 2 + 10 + 1 + 10 = 61
```

See also [Sidekiq in Practice 4: Database Connections](https://gist.github.com/nateberkopec/2d1fcf77dc61e747438252e3895badf0).

## Heroku CLI

Use [pg-extras](https://github.com/heroku/heroku-pg-extras) add-on for Heroku CLI to get insights
on the state of the DB:

```sh
$heroku plugins:install heroku-pg-extras
```

## Backups

Heroku provides _continuous protection_ out-of-the-box and requires no setup.
This protection only allows to rollback as far as for **4 days ago**.

We schedule an additional backup daily at **4am EST**: daily backups retain for 7 days and
weekly backups retain for 4 weeks.

Useful commands:

```sh
// list of existing backups
heroku pg:backups -a my-app

// list of scheduled backups
heroku pg:backups:schedules -a my-app
```

See more in [the docs](https://devcenter.heroku.com/articles/heroku-postgres-backups#scheduling-backups).

## Maintenance

You can check the data of the next required maintenance and the maintenance window settings by running
`heroku pg:info` command.

The current window: Mondays 08:00 to 12:00 **UTC**.

See also [docs](https://devcenter.heroku.com/articles/heroku-postgres-maintenance#setting-a-maintenance-window).

## Troubleshooting 🔥

If you see a lot of DB related exceptions and/or increased query times,
first thing to check is **locks and long-running queries**.

The following commands could be helpful:

```sh
# List of locks
heroku pg:locks

# Blocked queries
heroku pg:blocking

# Runing queries
heroku pg:calls

# Slow queries
heroku pg:outliers

# Processes
heroku pg:ps
```

If you need more granular control, you can connect to the database via `psql`:

```sh
heroku pg:psql
```

Useful queries:

```sql
-- show running queries
SELECT pid, age(query_start, clock_timestamp()), usename, query
FROM pg_stat_activity
WHERE query != '<IDLE>' AND query NOT ILIKE '%pg_stat_activity%'
ORDER BY query_start desc;

-- kill running query
SELECT pg_cancel_backend(procpid);

-- kill idle query
SELECT pg_terminate_backend(procpid);
```

Resources:

- [Lock Monitoring](https://wiki.postgresql.org/wiki/Lock_Monitoring)
- [Index Maintenance](https://wiki.postgresql.org/wiki/Index_Maintenance)
