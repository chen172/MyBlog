---

layout: post
title: "Understanding the Ruby Gem Rack `config.ru`"

---

### 1. Example: `test_rack_use.rb` (from [this link](https://github.com/chen172/rack/blob/master/my_own_test/test_rack_use.rb))

Let's start by creating a basic Rack application using middleware.

1. First, **rename** `test_rack_use.rb` to `config.ru`:

   ```bash
   mv test_rack_use.rb config.ru
   ```

2. To run the app, use the `rackup` command:

   ```bash
   rackup
   ```

Here’s the content of `config.ru`:

```ruby
class Middleware
  def initialize(app)
    puts "In initialize"
    @app = app
  end

  def call(env)
    puts "In call"
    env["rack.some_header"] = "Setting an example"
    @app.call(env)
  end
end

puts "Before Middleware"
use Middleware
puts "After Middleware"
run lambda { |env| [200, { "content-type" => "text/plain" }, ["OK"]] }
```

This code demonstrates how to set up a simple Rack application with custom middleware.

### 2. What is a `lambda` in Ruby?

In Ruby, `lambda` is a method provided by the `Kernel` module. A `lambda` is essentially an anonymous function (or closure) that can be called with arguments. It’s similar to a `Proc`, but with a few differences in behavior, particularly in how it handles arguments.

You can learn more about `lambda` in the official [Ruby documentation](https://docs.ruby-lang.org/en/master/Kernel.html#method-i-lambda).

#### Key points about `lambda`:

* It behaves like a method.
* It checks the number of arguments passed to it (unlike `Proc`).

In the example above, the `lambda` provides a simple Rack response.

### 3. Understanding `use` and `run` in Rack

Both `use` and `run` are methods provided by `Rack::Builder`, which is used to define a Rack application. These methods are part of the **DSL (Domain-Specific Language)** that Rack provides to configure middleware and define how the application handles requests.

* **`use`** is used to specify middleware in the application stack.
* **`run`** is used to define the final application endpoint, where a response is returned for incoming requests.

For more details, you can explore the source code of these methods:

* `use` is defined in the [Rack::Builder source](https://github.com/chen172/rack/blob/0b8d8394382a1d82889848b5c72531e0aa9403bf/lib/rack/builder.rb#L158).
* `run` is defined in the [Rack::Builder source](https://github.com/chen172/rack/blob/0b8d8394382a1d82889848b5c72531e0aa9403bf/lib/rack/builder.rb#L183).

These methods allow you to chain middleware in a flexible way and define how Rack handles requests and responses.

### 4. Tips for Navigating Code on GitHub

If you're reading through Rack's code or any open-source Ruby project on GitHub, you might want to easily navigate through definitions of functions, classes, or methods. GitHub’s code navigation feature can help you quickly find and link to the definitions.

For more information on how to use this feature, check out the official [GitHub documentation](https://docs.github.com/en/repositories/working-with-files/using-files/navigating-code-on-github).

#### Troubleshooting Code Navigation:

If you’re having trouble with code navigation on GitHub, keep these tips in mind:

* **Active branches only**: Code navigation only works on active branches, so make sure to push to the branch you're working on.
* **Repository size**: Repositories with fewer than 100,000 files support code navigation.
* **Unsupported languages**: Some languages may not be supported for code navigation.
