# honeybadger-elixir [![Build Status](https://travis-ci.org/honeybadger-io/honeybadger-elixir.svg?branch=master)](https://travis-ci.org/honeybadger-io/honeybadger-elixir)

Elixir Plug, Logger and client for the :zap: [Honeybadger error notifier](https://www.honeybadger.io/).

## Try it out

To deploy a sample Phoenix application which uses this library to report errors
to Honeybadger.io:

[![Deploy](https://www.herokucdn.com/deploy/button.png)](https://heroku.com/deploy?template=https://github.com/honeybadger-io/crywolf-elixir)

Don't forget to destroy the Heroku app after you're done so that you aren't
charged for usage.

You can also [download the sample
application](https://github.com/honeybadger-io/crywolf-elixir) and run it locally.

## Installation

Add the Honeybadger package to `deps/1` and `application/1` in your
application's `mix.exs` file and run `mix do deps.get, deps.compile`

```elixir
defp application do
 [applications: [:honeybadger, :logger]]
end

defp deps do
  [{:honeybadger, "~> 0.1"}]
end
```

## Configuration

By default the environment variable `HONEYBADGER_API_KEY` will be used to find
your API key to the Honeybadger API. If you would like to specify your key or
any other configuration options a different way, you can do so in `config.exs`:

```elixir
config :honeybadger,
  api_key: "sup3rs3cr3tk3y"
```

Here are all of the options you can pass in the keyword list:

| Name         | Description                                                               | Default |
|--------------|---------------------------------------------------------------------------|---------|
| api_key      | Your application's Honeybadger API key                                    | System.get_env("HONEYBADGER_API_KEY"))` |
| app          | Name of your app's OTP Application as an atom                             | Mix.Project.config[:app] |
| use_logger   | Enable the Honeybadger Logger for handling errors outside of web requests | false |
| exclude_envs | Environments that you want to disable Honeybadger notifications           | [:dev, :test] |
| hostname     | Hostname of the system your application is running on                     | :inet.gethostname |
| origin       | URL for the Honeybadger API                                               | "https://api.honeybadger.io" |
| project_root | Directory root for where your application is running                      | System.cwd |

## Usage

The Honeybadger package can be used as a Plug alongside your Phoenix
applications, as a logger backend, or as a standalone client for sprinkling in
exception notifications where they are needed.

## Phoenix and Plug

The Honeybadger Plug adds a
[Plug.ErrorHandler](https://github.com/elixir-lang/plug/blob/master/lib/plug/error_handler.ex)
to your pipeline. Simply `use` the `Honeybadger.Plug` module inside of a Plug
or Phoenix.Router and any crashes will be automatically reported to
Honeybadger. It's best to `use Honeybadger.Plug` **after the Router plugs** so that
exceptions due to non-matching routes are not reported to Honeybadger.

```elixir
defmodule MyPlugApp do
  use Plug.Router
  use Honeybadger.Plug
  
  [... the rest of your plug ...]
end

defmodule MyPhoenixApp.Router do
  use Crywolf.Web, :router
  use Honeybadger.Plug
  
  pipeline :browser do
    [...]
  end
end
```

## Logger

Just set the `use_logger` option to `true` in your application's `config.exs`
and you're good to go! Any
[SASL](http://www.erlang.org/doc/apps/sasl/error_logging.html) compliant
processes that crash will send an error report to the `Honeybadger.Logger`.
After the error reaches the logger we take care of notifying Honeybadger for
you!

## Standalone Client

Use the `Honeybadger.notify/2` macro to send exception information to the
collector API.  The first parameter is the exception and the second parameter
is the context/metadata.

```elixir

try do
  File.read! "this_file_really_should_exist_dang_it.txt"
rescue
  exception ->
    require Honeybadger
    context = %{user_id: 5, account_name: "Foo"}
    Honeybadger.notify(exception, context)
end
```

## Setting Context

`Honeybadger.context/1` is provided for adding extra data to the notification that gets sent to Honeybadger. You can make use of this in places such as a Plug in your Phoenix Router or Controller to ensure useful debugging data is sent along.

```elixir
def MyPhoenixApp.Controller
  use MyPhoenixApp.Web, :controller

  plug :set_honeybadger_context

  def set_honeybadger_context(conn, _opts) do
    user = get_user(conn)
    Honeybadger.context(user_id: user.id, account: user.account.name)
    conn
  end
end
```

## Changelog

See https://github.com/honeybadger-io/honeybadger-elixir/releases

## Contributing

If you're adding a new feature, please [submit an
issue](https://github.com/honeybadger-io/honeybadger-elixir/issues/new) as a
preliminary step; that way you can be (moderately) sure that your pull request
will be accepted.

### To contribute your code:

1. Fork it.
2. Create a topic branch `git checkout -b my_branch`
3. Commit your changes `git commit -am "Boom"`
3. Push to your branch `git push origin my_branch`
4. Send a [pull request](https://github.com/honeybadger-io/honeybadger-elixir/pulls)

### License

This library is MIT licensed. See the
[LICENSE](https://raw.github.com/honeybadger-io/honeybadger-elixir/master/LICENSE)
file in this repository for details.
