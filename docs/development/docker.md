# Docker for Development

Source: [Ruby on Whales: Dockerizing Ruby and Rails development](https://evilmartians.com/chronicles/ruby-on-whales-docker-for-ruby-rails-development).

## Installation

- `docker` and `docker-compose` installed.

For MacOS just use [official app](https://docs.docker.com/engine/installation/mac/).

- [`dip`](https://github.com/bibendi/dip) installed. Since our development `docker-compose.yml` is not located in the root (there is already a production one), using Docker without Dip wouldn't be a pleasure.

You can install `dip` either as Ruby gem:

```sh
gem install dip
```

Or using Homebrew:

```sh
brew tap bibendi/dip
brew install dip
```

Or by downloading a binary (see [releases](https://github.com/bibendi/dip/releases)):

```sh
curl -L https://github.com/bibendi/dip/releases/download/v3.8.3/dip-`uname -s`-`uname -m` > /usr/local/bin/dip
chmod +x /usr/local/bin/dip
```

## Provisioning

When using Dip it could be done with a single command:

```sh
dip provision
```

After this command finishes, you should be able to run `dip rspec` and see all tests passed (except from skipped and pending).

## Developing with Dip

### Useful commands

```sh
# run full application stack
dip up rails webpacker

# run rails console
dip rails c

# run rails server with debugging capabilities (i.e., `binding.pry` would work)
dip rails s

# run webpacker in the background
dip up -d webpacker
# `-d` - mean that service will run in detached (background) mode

# run migrations
dip rails db:migrate

# pass env variables into application
dip VERSION=20100905201547 rails db:migrate:down

# simply launch bash within app directory (with dependencies up)
dip sh

# or launch bash without any deps up (e.g., without database)
dip bash

# Additional commands

# update gems or packages
dip bundle install
dip yarn install

# run psql console
dip psql

# run tests
# TIP: `dip rspec` is already auto prefixed with `RAILS_ENV=test`
dip rspec spec/path/to/single/test.rb:23

# shutdown all containers
dip down
```

### Development Flows

One way to develop the app via `dip` it to launch the `sh` service and run all the commands
from inside it:

```sh
$ dip sh
root@aef12:/app# be rspec
```

**NOTE:** `be` is an alias for `bundle exec` which is available within the container.

Another way is to run `dip <smth>` for every interaction. If you prefer this way and use ZSH, you can reduce the typing
by integrating `dip` into your session:

```sh
$ dip console | source /dev/stdin
# no `dip` prefix is required anymore!
$ rails c
Loading development environment (Rails 6.0.0)
pry>
```

## Image Versioning

Every time you change the `Dockerfile` contents, you **must** increase the version number in the `docker-compose` file using the following rules:

- Increase the patch version if a bug has been fixed.
- Increase the minor version if a dependency has been added or upgraded (e.g. new Ruby version).
- Increase the major version only if the app stack has changed drastically (e.g. switched from fullstack app to API only, or from Ruby to Haskell).

## Docker Tips

**TIP #1**: Docker doesn't cleanup after itself well, so you have to do it manually.

```sh
# to monitor disk usage
docker system df

# to stop and clean application related containers
docker-compose -f .dockerdev/docker-compose.yml down

# and with volumes (useful to reset databases state)
docker-compose  -f .dockerdev/docker-compose.yml down --volumes

# or even clean the whole docker system (it'll affect other applications too!)
docker system prune
```

## TroubleShooting

If webpack server suddenly stops to watch files in Docker for Mac, we should just restart the whole Docker for Mac.
