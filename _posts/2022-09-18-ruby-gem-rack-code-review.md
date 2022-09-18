---
layout: post
title: "ruby gem rack code review"
---

### 1. test_rack_use.rb from [here](https://github.com/chen172/rack/blob/master/my_own_test/test_rack_use.rb)
```ruby
# rename to config.ru
## mv test_rack_use.rb config.ru
# run app
## rackup

class Middleware
  def initialize(app)
    puts "in initializee"
    @app = app
  end

  def call(env)
    puts "in call"
    env["rack.some_header"] = "setting an example"
    @app.call(env)
  end
end

  puts "before Middleware"
  use Middleware
  puts "after Middleware"
  run lambda { |env| [200, {"Content-Type" => "text/plain"}, ["OK"]]}
  ```

### 2. The meaning of ```lambda```
```lambda``` is ruby module kernel method, you can find doc from https://docs.ruby-lang.org/en/master/Kernel.html#method-i-lambda.

### 3. The meaning of ```use``` and ```run```
```use``` and ```run``` is rack method, you can find the code from [here](https://github.com/chen172/rack/blob/0b8d8394382a1d82889848b5c72531e0aa9403bf/lib/rack/builder.rb#L158) and [here](https://github.com/chen172/rack/blob/0b8d8394382a1d82889848b5c72531e0aa9403bf/lib/rack/builder.rb#L183).

### Tips
Github automatically enable code navigation can show and link definitions of a named entity corresponding to a reference to that entity.

You can find doc from https://docs.github.com/en/repositories/working-with-files/using-files/navigating-code-on-github.

##### Troubleshooting code navigation

If code navigation is enabled for you but you don't see links to the definitions of functions and methods:
* Code navigation only works for active branches. Push to the branch and try again.
* Code navigation only works for repositories with fewer than 100,000 files.
