# Heroku

Organization home page: `<url to Heroku dashboard>`

## Formation

We have three _services_ (dyno types): `web`, `worker` (Sidekiq) and `scheduler` ([Schked](https://github.com/bibendi/schked)).

**NOTE:** we use Heroku's [Preboot](https://devcenter.heroku.com/articles/preboot) feature for
["blue-green" deployment](https://martinfowler.com/bliki/BlueGreenDeployment.html).

## CLI

Heroku CLI provides helps you to manage the application directly from the terminal.

[Installation instructions](https://devcenter.heroku.com/articles/heroku-cli#download-and-install).

### Useful commands

**NOTE:** when working with multiple apps for the same repo (e.g. staging, review apps)
add `-a <app-name>` option to every command to specify the app you want to perform
the action against.

#### Logs

```sh
# stream logs for the app
heroku logs -t

# stream logs for `web` dynos only
heroku logs -t -d web
```

### Processes/Dynos

```sh
# list processes
heroku ps

# stop/start/restart
heroku ps:start <dyno_id>
heroku ps:restart <dyno_id>
heroku ps:stop <dyno_id>
```

#### Info

```sh
# get the add-ons information
heroku addons
```

#### Commands

Run arbitrary command using [one-off](https://devcenter.heroku.com/articles/one-off-dynos) dyno (e.g. Rake task):

```sh
heroku run "<cmd>"
```

## Resources

- [Big on Heroku](https://evilmartians.com/chronicles/big-on-heroku-scaling-fountain-without-losing-a-drop)
