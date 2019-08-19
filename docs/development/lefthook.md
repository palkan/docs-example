# Hooks & LeftHook

To automate local development we use Git hooks managed by [LeftHook](https://github.com/Arkweid/lefthook).

**NOTE:** this doc is valid for lefthook v0.3.x (check your version with the `lefthook version` command).

## Installation

- Install lefthook (see [Readme](https://github.com/Arkweid/lefthook) for instructions).

For example, via Hombrew:

```sh
brew install Arkweid/lefthook/lefthook
```

Or build from source (if you have Golang runtime installed):

```sh
go get github.com/Arkweid/lefthook
```

- Run the following commands to install hooks:

```sh
# install all custom scripts from `.lefthook` folder
$ lefthook install
```

### Usage

The existing hooks assumes that you have Ruby and [Docker](./docker.md) installed.

You can modify the hooks behavior and skip some hooks locally or change the command
using `lefthook-local.yml`.

For example, to run RuboCop hook on local machine put the following into your:

```yml
# lefthook-local.yml
pre-commit:
  rubocop:
    runner: "bundle exec rubocop --safe-auto-correct {staged_files} && git add {staged_files}"
```

Or to lint and fix JS files using local:

```yml
# lefthook-local.yml
pre-commit:
  rubocop:
    runner: "yarn lint:fix && git add {staged_files}"
```

This configuration will be merged with the default one (from `lefthook.yml`).

Another example is disabling backend specific checks:

```yml
# lefthook-local.yml
exclude_tags:
  - backend
```

### Skipping hooks

Use the standard Git option `--no-verify` to skip `pre-commit` and `commit-msg` hooks:

```sh
git commit -m "..." --no-verify
```

Use `LEFTHOOK_EXCLUDE=<tag>` to exclude only the specific tags or use `LEFTHOOK=0` to disable all LeftHook hooks.
