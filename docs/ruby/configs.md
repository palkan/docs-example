# Config Example

We're using [anyway_config](https://github.com/palkan/anyway_config) to deal with configuraiton.

## Example

Suppose that we want to add credentials for AWS S3.

We need the following _secrets_: `access_key_id`, `secret_access_key`, `region`, `bucket`.

First, we create a configuration class for this service:

```ruby
# app/configs/aws_config.rb
class AwsConfig < ApplicationConfig
  attr_config :access_key_id, :secret_access_key, :bucket,
              # we can set default right here
              region: "us-east-1"
end
```

We can initialize this config in the `application.rb` file:

```ruby
config.aws = AwsConfig.instance
```

The config is populated using the following data sources (in the provided order):

- `config/aws.yml` file (environment-aware)
- `config/credentials/<environment>.enc` (if any)
- `config/credentials.local.enc` (personal credentials set)
- environment variables (`AWS_ACCESS_KEY_ID`, `AWS_SECRET_ACCESS_KEY`, etc.)

For example, in `aws.yml` we can keep the following data:

```yml
test:
  # ideally, we shouldn't use S3 in tests at all;
  # but if we do, we must mock the network calls and
  # use these values in fixtures
  access_key_id: "aws-test-key"
  secret_access_key: "aws-test-key"
  bucket: "kin-test"

development:
  bucket: "kin-dev"

production:
  # it's ok to store the default production bucket as plain text
  bucket: "kin-production"

# Note that we also have a staging env
staging:
  bucket: "kin-staging"
```

All the sensitive information goes to Rails Credentials:

```yml
# config/credentials/production.enc
# Add settings under config name ("aws")
aws:
  access_key_id: <super-secret-value>
  secret_access_key: <another-secret-value>
```