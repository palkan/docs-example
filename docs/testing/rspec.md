# Testing with RSpec

This guide contains recommendations on writing tests with RSpec.

## Organizing test files

Test files structure should follow the app files structure, i.e., tests for the class defined in `app/something/klass.rb`
MUST be defined in `spec/something/klass_spec.rb`.

That helps to identify the relevant code quickly.

Store all non-code artifacts in the `spec/fixtures` folder. Group them by context/domain/any-other-self-descriptive-way.

Store all supporting files (helper, shared examples and contexts) in the `spec/support` folder. This folder is loaded automatically, so you don't need to require these files explicitly.

## `describe` and `context`

Always `describe` at the top-level.

Use class name under test as the description for the top-level `describe`:

```ruby
# bad
describe "deal generator" do; end

# good
describe Deal::Generator do; end
```

Use nested `describe` to group examples by the method under test. Use the following pattern for descriptions:

- for instance methods: `#<method_name>` (e.g., `describe "#sync_to_pg"`)
- for class methods: `.<method_name>` (e.g., `describe ".call"`).

You can omit the method grouping if the class has the only one method (e.g., for callable service objects you don't need to add `describe ".call"` context).

For controller and request specs use the following naming:

- for controllers: `"<METHOD> #<action>"` (e.g., `describe "GET #index"`)
- for requests: `"<METHOD> <path>"` (e.g., `describe "PATCH /api/v2/entity/:id"`).

Use contexts to share setup between multiple examples. Prefer to start context description with:

- `"when <smth>"` if you want to share hooks (`before`, `before_all`)
- `"with <smth>"` when you override previously defined `let` definitions.

## Hooks and memoization

**Never use `before(:all)`**

It doesn't clean up after itself and leads to flaky tests (e.g., when something got left in DB).

Use [`before_all`](https://test-prof.evilmartians.io/#/before_all?id=before-all) instead.

Consider using [`let_it_be`](https://test-prof.evilmartians.io/#/let_it_be) instead of `let!`—it will make tests faster.
You can also use `let_it_be` instead of `let` if you do not override it in nested contexts.

## Shared contexts

Consider extracting setup used in many (not few) test files into a [shared context](https://relishapp.com/rspec/rspec-core/docs/example-groups/shared-context).

Put _global_ shared contexts into `spec/support/shared_contexts` and domain-specific shared contexts into `spec/support/<domain>/<type>_context.rb`. For example:

- `sidekiq:inline` context should be stored in `spec/support/shared_contexts`
- `radix:service` context should be stored in `spec/support/radix/service_context.rb`

Name contexts using the following pattern where applicable: `<group>:<type>` (see the examples above).

Use tags to include contexts:

```ruby
shared_context "sidekiq:inline" do
  # ...
end

RSpec.configure do |config|
  config.include_context "sidekiq:inline", sidekiq: :inline
end

# and in test
it "depends on background job", sidekiq: :inline do; end
```

This approach has the following benefits:

- Ability to include contexts to examples (not only groups)
- Ability to filter specs by context (i.e., `rspec --tag twilio`).

## Aggregating examples

In general, follow the "one expectation per example" rule. But when the setup is _heavy_ and/or expectations are
independent, consider group them either by using specific matchers or by using [`:aggregate_failures` tag](https://relishapp.com/rspec/rspec-core/docs/expectation-framework-integration/aggregating-failures).

For example:

```ruby
# bad
it { is_expected.to be_success }
it { is_expected.to have_header("X-TOTAL-PAGES", 10) }
it { is_expected.to have_header("X-NEXT-PAGE", 2) }

# good
it "returns the second page", :aggregate_failures do
  is_expected.to be_success
  is_expected.to have_header("X-TOTAL-PAGES", 10)
  is_expected.to have_header("X-NEXT-PAGE", 2)
end

# bad
it { expect(subject.name).to eq "John" }
it { expect(subject.last_name).to eq "Green" }

# good
specify do
  expect(subject).to have_attributes(
    name: "John",
    last_name: "Green"
  )
end
```

## Mocking HTTP APIs with VCR

Whenever you want to stub outgoing HTTP requests in your tests (e.g., testing integration with third-party services),
consider using [VCR](https://github.com/vcr/vcr).

Add the following metadata to your example to record the outgoing request:

```ruby
it "makes request to CoolService API", vcr: {cassette_name: "cool_service", record: :once} do
  # do smth
end
```

After running this test you'll see a new file created in the `spec/fixtures/vcr` folder called `cool_service.yml`.
Next time you will run this test this file will be used to stub the API request.

Read more in the [VCR docs](https://relishapp.com/vcr/vcr).

## Using JSON fixtures for HTTP payloads (_aka_ `metadata`)

There are many places when we test the processing of the previously stored JSON _metadata_: from webhooks or external APIs.

Since this data is _external_ (we do not control it), we should treat it the same way we do with HTTP calls: mock using fixtures.

Capture the payloads (e.g., via VCR or manually) and store the responses in `spec/fixtures/<service-name>/<payload>.json` files.

We have a specific helper to access this fixtures in tests—`json_fixture`.
It returns a parsed JSON fixture stored in `spec/fixtures` by it's path:

```ruby
let(:data) { json_fixture("fa/payment") }

# you can override some keys by providing the second optional arg
let(:data) { json_fixture("fa/payment", id: "testID") }
```

The return value is a hash with indifferent access, so you can access values by using both strings and symbols.

## API requests specs

We include the `requests:api:v2` (`spec/shared_contexts/api_v2_requests.rb`) shared context to all API V2 specs automatically.
This shared context contains the common setup (e.g., headers, signed in user) and assumes the following convention is used:

```ruby
describe "GET /api/v2/some/thing" do
  # user (admin) is already created and signed in

  # Define the request under test.
  # NOTE: you must provide headers for JSON API. The default `headers`
  # value contains them
  let(:request) { get "/api/v2/some/thing", headers: headers }

  # The shared context defines the request's response as a subject,
  # so we can write assertions as follows
  specify { is_expected.to have_http_status(200) }

  # If you need to add custom headers, override the `headers` value.
  # You can access the default headers via `default_headers` value,
  # so you shouldn't worry about the utility JSON API headers at all
  let(:headers) { default_headers.merge("x-custom-header" => 123) }

  # If you want to access the JSON body, you can use `json_body` method
  # (it automatically performs the request if it hasn't been performed yet)
  it "has known JSON body" do
    expect(json_body).to include("a" => 1)
  end

  # Use `params` value to provide a query string (it's an empty string by default)
  let(:params) { "?sort=created_at" }
  let(:request) { get "/api/v2/resources#{params}", headers: headers }

  # Use `payload` value to provide a request body
  let(:payload) { {data: {type: "resources", attributes: {}}} }
  # NOTE: we must JSON encode payload explicitly
  let(:request) { post "/api/v2/resources", params: payload.to_json, headers: headers }
end
```

NOTE: the result of the request (and its `json_body`) are memoized, thus, even if you call `request` or `subject`
twice, only the first HTTP request would be made.
Sometimes it might be useful to perform the same request multiple times within the same test example (e.g., when testing with [`n_plus_one_control`][n_plus_one_control]). For that, you can use special `request!` and `json_body!` methods:

```ruby
context "N+1", :n_plus_one do
  warmup { subject }

  populate { |n| create_list(:something, n) }

  specify do
    # NOTE: here we use `request!`, 'cause this block is evaluated multiple times
    expect { request! }
      .to perform_constant_number_of_queries.with_scale_factors(0, 1)
  end
end
```

## Resources/Links

- [Better Specs](http://www.betterspecs.org)
- [`n_plus_one_control`][n_plus_one_control]

[n_plus_one_control]: https://github.com/palkan/n_plus_one_control
