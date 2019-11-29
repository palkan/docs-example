# Incident Response Plan

If you found that something is wrong with the app (high error rate, non-responsiveness, high request latency, whatever), first thing to do is to **inform the team**.

**Then, check logs and exceptions Sentry to get an idea what could go wrong.**

Take a look at the [Datadog](https://app.datadoghq.com) dashboards and Heroku Metrics.

In most cases outages are caused by:

- the wrong code deployed to production (tests and reviews do not guarantee 100% safety)
- _external_ systems failures (3rd-party APIs, databases, etc.).

## Suspect #1: _bad code_

If the outage is caused by the newly pushed release, the first thing to do is **rollback to the previous working release**:

- Get the list of the recent releases (see [docs](https://blog.heroku.com/releases-and-rollbacks)):

```sh
$ heroku releases

Rel   Change                   By                    When
----  ----------------------   -------------------   -------------
v52   Config add AWS_S3_KEY    someone@example.dev       5 minutes ago
v51   ...
```

- Rollback to the previous version **if possible**\*:

```sh
heroku rollback v51
```

\* There are some situations when it's not possible to rollback the code, e.g.: you've changed the DB schema (**you should never do that**) or updated the API and released a new client (=mobile app) depending on this new API (it's better not to do that as well).

- Make sure you're not deploying new releases (consider turning off auto-deploy during the investigation); **NOTE:** there is no way to stop the deployment in progress on Heroku, wait for it to complete and roll it back.

- If after rolling back the application returns back to a healthy state, you might consider reverting the merge commit with the _bad code_ (if it's not clear what's wrong with it and you don't want to block other deployments).

## Suspect #2: _external system failure_

- [Postgres troubleshooting](../infrastructure/postgres.md?id=troubleshooting-🔥).
- Anything else?

## Maintenance mode

If rolling back or pushing a quick fix is not possible, **the team** might consider putting the
application into a _maintenance mode_ (NOTE: it makes sense to do that only if the failure affects (almost) all of the users).

You can do that either from the Heroku console or using a Heroku CLI:

```sh
$ heroku maintenance:on

# and turn it off later by
$ heroku maintenance:off
```
