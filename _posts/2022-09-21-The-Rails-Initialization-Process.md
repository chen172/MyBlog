---
layout: post
title: "The Rails Initialization Process"
---

### 1. Run ```rackup```, start ```config.ru``` file from [here](https://gitlab.com/rluna-gitlab/gitlab-ce/-/blob/master/config.ru)
```ruby
# frozen_string_literal: true

# This file is used by Rack-based servers to start the application.

require ::File.expand_path('config/environment', __dir__)

warmup do |app|
  client = Rack::MockRequest.new(app)
  client.get('/')
end

map ENV['RAILS_RELATIVE_URL_ROOT'].presence || "/" do
  use Gitlab::Middleware::ReleaseEnv
  run Gitlab::Application
end
```

### 2. Then we run into ```config/environment.rb``` file from [here](https://gitlab.com/rluna-gitlab/gitlab-ce/-/blob/master/config/environment.rb)
```ruby
# frozen_string_literal: true

# Load the Rails application.
require_relative 'application'

# Initialize the Rails application.
Rails.application.initialize!
```

### 3. Run into ```config/application.rb``` file from [here](https://gitlab.com/rluna-gitlab/gitlab-ce/-/blob/master/config/application.rb)
In this file:
* load the railties needed
* define the configuration for the ```Rails::Application```

### 4. Back to ```config/environment.rb``` 
Run ```Rails.application.initialize!``` code

### 5. Turn to loading initializers
Run Ruby file stored under ```config/initializers``` directory from [here](https://gitlab.com/rluna-gitlab/gitlab-ce/-/tree/master/config/initializers)

### 6. Traversing all the class ancestors looking for those that respond to an initializers method
Do railtie initializers

#### Reference:
1. https://guides.rubyonrails.org/initialization.html
2. https://guides.rubyonrails.org/configuring.html#using-initializer-files
