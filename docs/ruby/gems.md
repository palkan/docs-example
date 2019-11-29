# Ruby Gems

This document contains the information about Ruby gems we use (not every one, but the crucial ones).

## Development Tools

### `rubocop`

[RuboCop](http://www.rubocop.org) is a Ruby code static analysis tool.

It helps to prevent syntax errors, simple bugs and enforce style guides.

Our RuboCop config is based on the [`standard`](https://github.com/testdouble/standard) gem and
also includes RSpec related cops and a few custom cops.

To run RuboCop use the following command:

```sh
bundle exec rubocop
```

To auto-format the code run:

```sh
bundle exec rubocop --fix-layout
```

### `brakeman`

We use [`brakeman`](https://brakemanscanner.org/) to detect security issues in the application code.

We run it on CI, but you can also run it locally:

```sh
bundle exec brakeman
```

Sometimes Brakeman generates false positives we want to ignore.
To do that run Brakeman locally in interactive mode:

```sh
bundle exec brakeman -I
```

### `bundle-audit`

We use [`bundler-audit`](https://github.com/rubysec/bundler-audit) to track security vulnerabilities in our dependencies.

It runs automatically on CI, no need to run it localy.

### `danger`

[Danger](https://danger.systems/ruby/) is a code review automation tool.

We run it on CI to enforce code styles and do some checks (e.g. security checks with `brakeman`).

Rules are stored in the `.danger/` folder: one file – one check.

The main `Dangerfile` contains some share logic and loads all the rules.

To add a new rule just add Ruby file to `.danger/` and use Danger DSL.

See also [Danger on Rails](https://evilmartians.com/chronicles/danger-on-rails-make-robots-do-some-code-review-for-you).

### `database_consistency`

This tool helps to keep AR models and database in the consistent state: checks for missing validations
or `null` constraints, missing indexes and foreign keys.

Run the check locally:

```sh
bundle exec rails db:consistency_check
```

**NOTE:** it's currently not stable enough to be added to CI (see [this comment](https://github.com/djezzzl/database_consistency/issues/7#issuecomment-483826193)). Just run it locally from time to time (e.g. after adding new tables/models).

## Frameworks

### `action_policy`

[Action Policy](https://actionpolicy.evilmartians.io/) is an _authorization_ framework (alternative to [CanCanCan](https://github.com/CanCanCommunity/cancancan) and kind of successor of [Pundit](https://github.com/varvet/pundit)).

Read more in the [documentation](https://actionpolicy.evilmartians.io/), or watch the [RailsConf talk](https://www.youtube.com/watch?v=NVwx0DARDis).

### `active_delivery` / `abstract_notifier`

[Active Delivery](https://github.com/palkan/active_delivery) and [Abstract Notifier](https://github.com/palkan/abstract_notifier) frameworks are used to unify the ways we notify users (via email or Push notifications).

Resources:

- [Crafting user notifications in Rails with Active Delivery](https://dev.to/evilmartians/crafting-user-notifications-in-rails-with-active-delivery-5cn6)

### `graphql-ruby`

[GraphQL](https://graphql.org) is a query language which we use for client-server communication.

We use [`graphql-ruby`](http://graphql-ruby.org) gem on backend side and [Apollo](https://www.apollographql.com) on client side.

Resources:

- [GraphQL on Rails](https://evilmartians.com/chronicles/graphql-on-rails-1-from-zero-to-the-first-query)

**NOTE:** we using the modern, object-oriented, API for `graphql-ruby`; so, there are many outdated post/tutorials out there–just skip them!

## Libraries

### `anyway_config`

We use [anyway_config](https://github.com/palkan/anyway_config) gem under the hood, which allows us to manage different sources of data transparently.

See the [configuration article](../ruby/configs.md) for more.

### `dry-initializer`

We use [Dry Initializer](https://github.com/dry-rb/dry-initializer) to standartize Service Objects (it's included into `ConnectBy::BaseService`), e.g.:

```ruby
class MyService < ConnectBy::BaseService
  # define params and options
  param :user
  option :some_type

  # define call method
  def call
    # here you can access `user` and `some_type`
  end
end

# Calling a service object
MyService.call(User.first, some_type: "foo")
```

### `jwt_sessions`

We use [JWT](https://jwt.io) for mobile client authentication backed by [`jwt_sessions`](https://github.com/tuwukee/jwt_sessions) gem.

Resources:

- [Rails API + JWT auth](https://blog.usejournal.com/rails-api-jwt-auth-vuejs-spa-eb4cf740a3ae)

### `schked`

[Schked](https://github.com/bibendi/schked) is a cron replacement. It runs as a lightweight, long-running Ruby process.

To run Schked locally use the following command:

```sh
bundle exec schked start
```

### `sorcery`

[Sorcery](https://github.com/sorcery/sorcery) is an authentication library for Rails app which provides passwords/tokens/sessions management.

Unlike `devise`, it doesn't contain tons of useless auto-generated code and provides only a bare mininum to build custom authentication system.

### `feature_toggles`

[Feature Toggles](https://github.com/bibendi/feature_toggles) This gem provides a mechanism for pending features that take longer than a single release cycle. The basic idea is to have a configuration file that defines a bunch of toggles for various features you have pending. The running application then uses these toggles in order to decide whether or not to show the new feature.

## ActiveRecord Extensions

We have multiple gems which extend ActiveRecord functionality (mostly for PostgreSQL specific features).

We use [`with_advisory_lock`](https://github.com/ClosureTree/with_advisory_lock) gem to add PostgreSQL [advisory locks](https://www.postgresql.org/docs/current/static/explicit-locking.html#ADVISORY-LOCKS) capabilities.

[This article](https://dev.to/evilmartians/the-silence-of-the-ruby-exceptions-a-railspostgresql-database-transaction-thriller-5e30) shows some real-life use cases for advisory locks.

We also use Postres as a [Full-text search feature](https://www.postgresql.org/docs/current/static/textsearch.html) along with [`pg_search`](https://github.com/Casecommons/pg_search) gem.

For cloning a complex structure of AR models we used [Clowne](https://github.com/palkan/clowne). [Documentation](https://evilmartians.com/chronicles/clowne-clone-ruby-models-with-a-smile).

We use [`postgresql_cursor`](https://github.com/afair/postgresql_cursor) for handling large Result Sets. See the [postgresql documentation](https://www.postgresql.org/docs/current/static/plpgsql-cursors.html) for more information.

We use [`activerecord-postgres_enum`](https://github.com/bibendi/activerecord-postgres_enum) to manage DB enum data types.

We use [`translate_enum`](https://github.com/shlima/translate_enum) to translate enums to human readable labels.

We use [`fx`](https://github.com/teoljungberg/fx) to manage DB functions and triggers.

Other noticeable PostgreSQL feature–unstructured data types (such as [arrays](https://www.postgresql.org/docs/current/static/arrays.html) and [JSONB](https://www.postgresql.org/docs/current/static/datatype-json.html)) are covered by [`pgrel`](https://github.com/palkan/pgrel) and [`store_attribute`](https://github.com/palkan/store_attribute/tree/master/lib) gems: the first one provides user-friendly API to query unstructured data with ActiveRecord, the second one adds an ability to type cast store values.

We use JSONB columns to store users and events tags. That allows us to efficiently query against tags (since JSONB is indexable) and avoid additional tables to connect resources to tags.

Most of the models have `extra` JSONB column, which is used to store some _meta_ information (which we don't need to use in queries). That's where we typically use `store_attribute` (e.g. to store boolean values in stores).

We use Arel for complex queries. Arel is a core part of ActiveRecord (existed as a separated gem until recently, now it's a part of Rails), it builds and manipulates _abstract syntax trees_ for SQL queries.

Arel (sometimes) is better than writing plain SQL, 'cause it's reusable, handles type casting, etc. But too much Arel could make code less readable and understandable.

Unfortunately, there is no good documentation for Arel. Some useful resources:

- [Arel Cheatsheet](https://devhints.io/arel)
- [Arel Cheatsheet 2](https://www.calebwoods.com/2015/08/11/advanced-arel-cheat-sheet/)
- [Scuttle: A SQL and Arel Editor](http://www.scuttle.io)
- [Common Table Expressions in ActiveRecord](https://sonnym.github.io/2017/06/05/common-table-expressions-in-activerecord-a-case-study-of-quantiles/) (_hardcore_)
