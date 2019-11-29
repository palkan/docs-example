# Using production DB dump in development

First, download the latest backup from Heroku:

```sh
dip bash -c "pg_dump -Fc --no-acl --no-owner -f tmp/latest.dump $(heroku pg:credentials:url FOLLOWER_DATABASE_URL -a my-cool-project | grep postgres://)"
```

Then, restore it to the `postgres-dump` container via the `rails` service:

```sh
# Run Postgres container for prod dump in the background
dip up -d postgres-dump

# Create databases inside postgres-dump container
DB=postgres-dump dip rake db:create

# Or re-create has been created previously
DB=postgres-dump dip rake db:drop db:create

# NOTE: if db:drop fails due to environment check, do the following
DB=postgres-dump dip rails runner "ActiveRecord::InternalMetadata.where(key: 'environment').update_all(value: 'development')"

# Restore the data from the dump into postgres-dump container
dip bash -c "pg_restore -O -d app_development -U postgres -h 'postgres-dump' tmp/latest.dump"
```

Now, you can use production database with any service:

```sh
# Ensure postgres-dump container is running
dip up -d postgres-dump

# Run rails console with production database
DB=postgres-dump dip rails c

# Run application with production database
DB=postgres-dump dip up webpacker rails sidekiq
```
