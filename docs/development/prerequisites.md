# Development Prerequisites

To start working on the project you need to obtain the following information first (otherwise you won't be able to do `rails s`):

1. Sidekiq Pro gems registry username and password.

_Could be found in the team's Keybase forlder._

Use this information to configure bundler:

```sh
bundle config --local gems.contribsys.com username:password
bundle config --local enterprise.contribsys.com username:password
```

1. `config/application.yml` and `config/master.key` file.

_Could be found in the team's Keybase folder._

After getting all of the above, you can start setting up the application (e.g., using [Docker](./docker.md))
