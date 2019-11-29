# Deployment Configuration

## Environment Variables

This table contains a list of the environment variables and where they come from.

NAME                       | REQUIRED     | SOURCE            | DESCRIPTION
---------------------------|--------------|-------------------|---------------
`RAILS_MASTER_KEY`         | ✅           | User              | **NOTE**: the key is stored in the Keybase team folder
`DATABASE_URL`             | ✅           | Heroku add-on     |
`REDIS_URL`                | ✅           | Heroku add-on     |
`WEB_CONCURRENCY`\*        |              | User              | Number of Puma workers (processes) (**default: 2**)
`RAILS_MAX_THREADS`\*      |              | User              | Used as a thread pool size for Puma and DB connection pool size (**default: 5**)
`SIDEKIQ_CONCURRENCY`\*    |              | User              | Set the concurrency (thread pool size) for Sidekiq
`SIDEKIQ_REDIS_POOL` \*    |              | User              | Set the size of the Sidekiq Redis pool

\* Be careful with this params and always consider external deps (such as databases) limits when changing them. See [Postgres](../infrastructure/postgres.md) and [Redis](../infrastructure/redis.md) docs.
