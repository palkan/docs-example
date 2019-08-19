# Docker for Development

## Installation

You need `docker` and `docker-compose` installed (for MacOS just use [official app](https://docs.docker.com/engine/installation/mac/)).

## Provisioning

Run the following commands to prepare your Docker dev env:

```sh
docker-compose build
docker-compose run runner ./bin/setup
```

It builds the Docker image, installs Ruby and NodeJS dependencies, creates database, run migrations and seeds.

You're all set! Now you're ready to code!

## Commands

* Running the app:

You can run the whole stack at once (Rails, Sidekiq, Webpacker(-s)) by running:

```sh
docker-compose up web
```

Or you can run each service separately:

```sh
# Running only Rails app and webpacker
$ docker-compose up rails webpacker

# Running Sidekiq
$ docker-compose up sidekiq
```

It could be useful to run Rails app without code reloading (e.g. if you need it just
as a local backend for mobile app and want it to be faster):

```sh
CODE_RELOAD=0 docker-compose up rails
```

* Running tests and other tools

You can spin a container to run tests:

```sh
RAILS_ENV=test docker-compose run rspec bundle exec rspec
```

A more flexible way is to run a `sh` session within a container and run commands
from inside:

```sh
$ docker-compose run runner
root@aef12:/app# be rspec
```

**NOTE:** `be` is an alias for `bundle exec` which is available within the container.

## Image Versioning

Every time you change the `Dockerfile` contents, you **must** increase the version number in the `docker-compose` file using the following rules:

* Increase the patch version if a bug has been fixed
* Increase the minor version if a dependency has been added or upgraded (e.g. new Ruby version)
* Increase the major version only if the app stack has changed drastically (e.g. switched from full-stack app to API only, or from Ruby to Haskell)

## Docker Tips

**TIP #1**: Docker doesn't cleanup after itself well, so you have to do it manually.

```sh
# to monitor disk usage
docker system df

# to stop and clean application related containers
docker-compose down

# and with volumes (useful to reset databases state)
docker-compose down --volumes

# or even clean the whole docker system (it'll affect other applications too!)
docker system prune
```

**TIP #2**: Use `--rm` flag with `run` command to automatically destroy the
container on exit, e.g. `docker-compose run --rm runner`.

**TIP #3**: Add system aliases:

```sh
alias dcr='docker-compose run --rm'
alias dcu='docker-compose up'
alias dcs='docker-compose stop'
alias dstats='docker stats --format "table {{.Name}}:\t{{.MemUsage}}\t{{.CPUPerc}}"'
```

**TIP #3**: Use [`dip`](./dip.md).
