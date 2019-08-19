# Docker on Dip Development Configuration

We have development [Docker configuration](./docker.md).

In addition to it you can also use [`dip`](https://github.com/bibendi/dip)–CLI utility for straightforward provisioning and interacting with an applications configured by `docker-compose`.

To install `dip` copy and run the command below:

```sh
gem install dip
```

## Usage

```sh
# provision application
dip provision

# run web app with all debuging capabilities (i.e. `binding.pry`)
dip rails s

# run rails console
dip rails c

# run webpacker
dip up -d webpacker
# `-d` - mean that service will run in detached (background) mode

# run migrations
dip rake db:migrate

# pass env variables into application
dip VERSION=20100905201547 rake db:migrate:down

# run sidekiq
dip up sidekiq

# simply launch bash within app directory
dip bash

# Additional commands

# update gems or packages
dip bundle install
dip yarn install

# run psql console
dip psql project_development
# where `project_development` is a database name. It might be `project_test`.

# run redis-cli
dip redis-cli

# run tests
# TIP: `dip rspec` is already auto prefixed with `RAILS_ENV=test`
dip rspec spec/path/to/single/test.rb:23

# run system tests
# webpacker should be running with test environment
RAILS_ENV=test dip up webpacker
dip rspec spec/system

# shutdown all containers
dip down
```

Starting from [version 3.5](https://github.com/bibendi/dip/releases/tag/v3.5.0), Dip can be easily integrated into ZSH shell, especially if used `agnostic` theme. So, there is allow us using all above commands without `dip` prefix. And it looks like we are not using Docker at all (really that's not true).

```sh
dip console | source /dev/stdin

rails c
rails s
rspec spec/test_spec.rb:23
psql kin_test
redis-cli
VERSION=20100905201547 rake db:migrate:down
rake routes | grep admin
```

But after we get out to somewhere from the project's directory, then all Dip's aliases will be cleared. And if we get back, then Dip's aliases would be restored.
