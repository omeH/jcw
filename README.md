# JCW &middot; [![Supporting](https://github.com/Cado-Labs/cado-labs-logos/blob/main/cado_labs_badge.png)](https://github.com/Cado-Labs/) &middot; [![Coverage Status](https://coveralls.io/repos/github/Cado-Labs/jcw/badge.svg?branch=gem-without-zeitwerk)](https://coveralls.io/github/Cado-Labs/jcw?branch=gem-without-zeitwerk) &middot; [![Gem Version](https://badge.fury.io/rb/jcw.svg)](https://badge.fury.io/rb/jcw)

Simple wrapper for the gem "jaeger-client" with simpler customization.

---

<p>
  <a href="https://github.com/Cado-Labs">
    <img src="https://github.com/Cado-Labs/cado-labs-logos/blob/main/cado_labs_supporting.svg" alt="Supported by Cado Labs" />
  </a>
</p>

---

## Installation

```ruby
gem 'jcw'
```

```shell
bundle install
# --- or ---
gem install jcw
```

```ruby
require 'jcw' 
```

## Usage

Create new initializer for your rails app:

UDP Sender(default):
```ruby
::JCW::Wrapper.configure do |config|
  config.service_name = "Service name"
  config.connection = { protocol: :udp, host: "127.0.0.1", port: 6831 }
  config.enabled = true
  config.tags = {
    hostname: "custom-hostname",
    custom_tag: "custom-tag-value",
  }
end

# Set middleware for wrapping all requests(gem RackTracer)
Rails.application.middleware.use(JCW::RackTracer)
```

TCP Sender:
```ruby
::JCW::Wrapper.configure do |config|
  config.service_name = "Service name"
  config.enabled = true
  config.subscribe_to = %w[process_action.action_controller start_processing.action_controller] # set ActiveSupport::Notifications namespaces
  config.connection = { protocol: :tcp, url: "http://localhost:14268/api/traces", headers: { key: "value" } }
  config.tags = {
    hostname: "custom-hostname",
    custom_tag: "custom-tag-value",
  }
end

# Set middleware for wrapping all requests
Rails.application.middleware.use(JCW::RackTracer)

# If you need send all logs with spans set on_finish_span and extend JaegerLoggerExtension
# Not recommended for UDP sender, because default max packet size is 65,000 bytes.
Rails.application.config.tap do |config|
  config.middleware.use(
  ::JCW::RackTracer,
  on_finish_span:
    -> (span) { ::JCW::JaegerLogger.current.logs.each { |log| span.log_kv(**log) } },
  )
  config.logger.extend(::JCW::JaegerLoggerExtension)
end
```
- `config.subscribe_to` - not recommended for UDP sender, because default max packet size is 65,000 bytes.

### Contributing
 
 - Fork it ( https://github.com/Cado-Labs/jcw )
 - Create your feature branch (`git checkout -b feature/my-new-feature`)
 - Commit your changes (`git commit -am '[feature_context] Add some feature'`)
 - Push to the branch (`git push origin feature/my-new-feature`)
 - Create new Pull Request

## License

Released under MIT License.

## Supporting

<a href="https://github.com/Cado-Labs">
  <img src="https://github.com/Cado-Labs/cado-labs-logos/blob/main/cado_labs_logo.png" alt="Supported by Cado Labs" />
</a>

## Authors

[Aleksandr Starovojtov](https://github.com/AS-AlStar)
