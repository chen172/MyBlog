---

layout: post
title: "Understanding the Rails Initialization Process"

---

### 1. Running `rackup` to Start the `config.ru` File

The Rails initialization process begins when you run the `rackup` command, which starts the application using the `config.ru` file. Here’s the content of a typical `config.ru` file, like the one from [GitLab](https://gitlab.com/rluna-gitlab/gitlab-ce/-/blob/master/config.ru):

```ruby
# frozen_string_literal: true

# This file is used by Rack-based servers to start the application.

require ::File.expand_path('config/environment', __dir__)

# Warm-up the application with a mock request to the root
warmup do |app|
  client = Rack::MockRequest.new(app)
  client.get('/')
end

# Set up the environment and start the application
map ENV['RAILS_RELATIVE_URL_ROOT'].presence || "/" do
  use Gitlab::Middleware::ReleaseEnv
  run Gitlab::Application
end
```

* The `require ::File.expand_path('config/environment', __dir__)` line loads the Rails environment configuration.
* The `warmup` block ensures the application is "warmed up" by making a mock request to the root route (`/`).
* The `map` block configures the base URL for the application, and the `run` block initializes the GitLab application by running `Gitlab::Application`.

### 2. Loading `config/environment.rb`

Next, the `config.ru` file loads the `config/environment.rb` file to initialize the environment. The content of `config/environment.rb` from [GitLab](https://gitlab.com/rluna-gitlab/gitlab-ce/-/blob/master/config/environment.rb) looks like this:

```ruby
# frozen_string_literal: true

# Load the Rails application
require_relative 'application'

# Initialize the Rails application
Rails.application.initialize!
```

* `require_relative 'application'` loads the main application configuration.
* `Rails.application.initialize!` initializes the entire Rails application, setting up the environment and configuration files.

### 3. Loading `config/application.rb`

The next step is the loading of the `config/application.rb` file. Here, the following things occur:

* The required **Railties** are loaded, which are the core components of Rails (such as ActiveRecord, ActionController, etc.).
* The configuration for the `Rails::Application` class is defined, which sets up the environment, middleware, and other settings.

Here’s an outline of what happens in `config/application.rb`:

```ruby
module YourApp
  class Application < Rails::Application
    # Add Railtie configuration here
    config.load_defaults 6.0  # Example for Rails 6
    config.time_zone = 'UTC'
    # Other application-wide settings
  end
end
```

### 4. Returning to `config/environment.rb`

After loading the application configuration, Rails calls `Rails.application.initialize!` to complete the application setup. This line finalizes the environment and prepares Rails for handling requests.

### 5. Loading Initializers

Once the application is initialized, Rails loads any custom initializers stored in the `config/initializers` directory. These files are executed in alphabetical order, allowing you to configure third-party libraries, set up application settings, or perform other setup tasks.

Here’s an example from [GitLab’s initializers](https://gitlab.com/rluna-gitlab/gitlab-ce/-/tree/master/config/initializers):

```ruby
# Example initializer file: config/initializers/custom_initializer.rb
SomeLibrary.configure do |config|
  config.api_key = ENV['API_KEY']
end
```

### 6. Running Railtie Initializers

As part of the initialization process, Rails also loads **Railtie** initializers. Railties are the foundational modules that allow Rails to interact with external frameworks and services. When the application is initialized, Rails traverses all class ancestors looking for classes that respond to the `initializer` method.

For example, the Rails framework and third-party gems use Railties to hook into the Rails initialization process and set up their required components.

### Reference Links:

1. [Rails Guides - Initialization Process](https://guides.rubyonrails.org/initialization.html)
2. [Rails Guides - Configuring Initializers](https://guides.rubyonrails.org/configuring.html#using-initializer-files)
