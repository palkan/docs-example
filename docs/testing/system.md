# System Tests

We use Rails System Tests (via RSpec integration) for integration/browser tests.

## Requirements

System tests require a browser (Chrome). Make sure you have Chrome and
[`chromedriver`](http://chromedriver.chromium.org/) installed if you want to run system tests locally.

When using [Docker](../development/docker.md) run the following command to start the container with Chrome browser
(headless by default):

```sh
dip up selenium

# or you can run rspec command which launches selenium as a dependency
dip rspec
```

## Writing specs

- Put tests under `spec/system` folder (the `type: :system` tag is added automatically).
- Require `system_helper.rb` at the top of the file (e.g., `require "system_helper.rb"`).
- Add `:js` tag if you need to interact with the _real_ browser (not just HTML).
- See `spec/support/feature_helper.rb` for custom Capybara helpers.

## Dealing with assets

For better system tests performance, run Webpack dev server in test env in the background:

```sh
RAILS_ENV=test dip up -d webpacker
```

**NOTE:** `RAILS_ENV=test` is important here.

You can also fallback to assets precompilation (which happens automatically if Webpack dev server is not running).

### Assets-related exceptions

In some situations, precompilation doesn't work (usually happens when you mix both dev server and precompilation approaches) and you may see, for example, "file not found" errors in tests.

To resolve the issue drop the `public/packs-test` folder and run test again.

## Debugging

### Connect to remote browser (Docker only)

We Selenium Docker image with built-in VNC server which allows you to connect to the
remote desktop to see what's going on inside (and even interact with the browser).

To enable the debug mode:

- Run Selenium container using `DEBUG=true` env flag:

```sh
DEBUG=true dip up selenium
```

- Connect to VNC using [vnc://locahost:5900](vnc://locahost:5900) (for MacOS use the built-in Screen Sharing app or just type the address in Search Spotlight) **NOTE:** if `localhost` doesn't work, try to connect to [vnc://0.0.0.0:5900](vnc://0.0.0.0:5900)
- Run tests using `HEADLESS=false` (or `HEADLESS=0`) env flag.

```sh
HEADLESS=0 dip rspec spec/system
```

## Resources

- [Capybara cheatsheet](https://devhints.io/capybara)
- [Running Rails 5 System Tests on Travis CI with Chromedriver](https://dev.to/aergonaut/running-rails-5-system-tests-on-travis-ci-with-chromedriver-4nm7)
