# Redis

## Limits vs. Concurrency

We use [Heroku Redis](https://elements.heroku.com/addons/heroku-redis) add-on.

You can check the current plan by running the following commands:

```sh
# get the list of add-ons
$ heroku addons
# => heroku-redis (redis-<id>)

# now get the add-on info
$ heroku addons:info "redis-<id>"
```

**NOTE:** when [Preboot](https://devcenter.heroku.com/articles/preboot) is enabled for Heroku app (enabled for production), the number of `web` dynos at a time could be doubled (during the _preboot_ phase). Thus we should add a multiplier (2) to the final formula.

The number of Redis connections required for Sidekiq to work could be calculated
using the following formula:

```
// Client (web/scheduler) Redis pool size

// we set client pool size to 3 for web dynos,
// 2 — preboot multiplier
web_connections = (web_dynos * (3 * web_workers) * 2)

// we set client pool size to 1 for scheduler
scheduler_connections = (scheduler_dynos * 1)

// Server (worker) has a reserved number of connections (5) and the pool equal to the concurrency value
worker_connections = (worker_dynos * (sidekiq_concurrency + 5))

// Additional connections to use in one-off dynos
one_off_connections = 10
```

For example, with two web dynos with `WEB_CONCURRENCY=2`, `RAILS_MAX_THREADS=5`, one worker with `SIDEKIQ_CONCURRENCY=10` and one scheduler we have:

```
2 * 2 * 3 * 2 + (10 + 5) + 1 + 10 = 50
```

See [Sidekiq Wiki](https://github.com/mperham/sidekiq/wiki/FAQ#how-to-calculate-the-number-or-redis-connection-used-by-sidekiq) for more.
