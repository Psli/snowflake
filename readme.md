# Snowflake

[![build](https://github.com/frm/snowflake/actions/workflows/build.yml/badge.svg)](https://github.com/frm/snowflake/actions/workflows/build.yml)

> This project was forked from [blitzstudios/snowflake] and its maintenance is
> focused on my personal and professional usage. Feel free to use it as is, and
> reach out through an issue or pull-request. I'll gladly consider your
> suggestions and contributions.

A scalable, decentralized Snowflake generator in Elixir.

## Usage

In your mix.exs file:

```elixir
def deps do
  [{:snowflake, github: "frm/snowflake", tag: "v1.2.0"}] # Replace with the new version tag once created
end
```

This library requires Elixir `~> 1.15`.

Ensure `:snowflake` is listed in your `application` function in `mix.exs` if you are using Elixir versions prior to 1.4 or if you need to explicitly manage application startup order:

```elixir
def application do
  [applications: [:snowflake]] # Or applications: [:logger, :runtime_tools, :snowflake] for Elixir < 1.4
end
```
For Elixir 1.4+ with Mix 1.4+, if `:snowflake` is listed in `:deps`, it's typically started automatically. However, explicitly listing it is fine.

Specify the nodes in your config. If you're running a cluster, specify all the nodes in the cluster that snowflake runs on.

- **nodes** can be Erlang Node Names, Public IPs, Private IPs, Hostnames, or FQDNs
- **epoch** should not be changed once you begin generating IDs and want to maintain sorting
- There should be no more than 1 snowflake generator per node, or you risk potential duplicate snowflakes on the same node.

```elixir
config :snowflake,
  nodes: ["127.0.0.1", :'nonode@nohost'],   # up to 1023 nodes
  epoch: 1142974214000  # don't change after you decide what your epoch is
```

Alternatively, you can specify a specific machine_id

```elixir
config :snowflake,
  machine_id: 23,   # values are 0 thru 1023 nodes
  epoch: 1142974214000  # don't change after you decide what your epoch is
```

You can also use the `adapter` option, which would allow you to use a separate
adapter to generate the Snowflakes. In my project's case, starting and killing the
Snowflake generator was causing unwanted delays in the test runs so I added a
`Snowflake.Adapter.Inline` adapter for testing purposes

```elixir
config :snowflake,
  adapter: Snowflake.Adapter.Inline # Only use this for testing purposes
```

Generating an ID is simple.

```elixir
Snowflake.next_id()
# => {:ok, 54974240033603584}
```

## Util functions

After generating snowflake IDs, you may want to use them to do other things.
For example, deriving a bucket number from a snowflake to use as part of a
composite key in Cassandra in the attempt to limit partition size.

Lets say I want to know the current bucket for an ID that would be generated right now:

```elixir
Snowflake.Util.bucket(30, :days)
# => 5
```

Or if I want to know which bucket a snowflake ID should belong to, given I are
bucketing by every 30 days.

```elixir
Snowflake.Util.bucket(30, :days, 54974240033603584)
# => 5
```

Or if I want to know how many ms elapsed from epoch

```elixir
Snowflake.Util.timestamp_of_id(54974240033603584)
# => 197588482172
```

Or if I want to know how many ms elapsed from computer epoch (January 1, 1970 midnight). We can use this to derive an actual calendar date.

```elixir
Snowflake.Util.real_timestamp_of_id(54974240033603584)
# => 1486669389497
```

## NTP

Keep your nodes in sync with [ntpd](https://en.wikipedia.org/wiki/Ntpd) or use
your VM equivalent as snowflake depends on OS time. ntpd's job is to slow down
or speed up the clock so that it syncs os time with your network time.

## Architecture

Snowflake allows the user to specify the nodes in the cluster, each representing a machine. Snowflake at startup inspects itself for Node, IP and Host information and derives its machine_id from the location of itself in the list of nodes defined in the config.

Machine ID is defaulted to **1023** if snowflake is not able to find itself within the specified config. It is important to specify the correct IPs / Hostnames / FQDNs for the nodes in a production environment to avoid any chance of snowflake collision.

## Benchmarks

Consistently generates over 60,000 snowflakes per second on Macbook Pro 2.5 GHz Intel Core i7 w/ 16 GB RAM.

```
Benchmarking snowflake...
Benchmarking snowflakex...

Name                 ips        average  deviation         median
snowflake       316.51 K        3.16 μs   ±503.52%        3.00 μs
snowflakex      296.26 K        3.38 μs   ±514.60%        3.00 μs

Comparison:
snowflake       316.51 K
snowflakex      296.26 K - 1.07x slower
```

## Differences from [blitzstudios/snowflake]

This is a list of differences based on the internal usage in several of my
projects. Most, if not all of these, have been made into pull requests. If and
when they have been merged, I urge everyone to use [blitzstudios/snowflake]:

- Added support for an inline adapter.
- [#5 - Adds semantic typing information](https://github.com/blitzstudios/snowflake/pull/5) - by [@Sonato](https://github.com/Sonato)
- [#8 - Run mix formatter](https://github.com/blitzstudios/snowflake/pull/8)
- [#9 - Remove deprecated Supervisor.Spec](https://github.com/blitzstudios/snowflake/pull/9)
- [#10 - import Config instead of the deprecated Mix.Config](https://github.com/blitzstudios/snowflake/pull/10)

## Attribution

Forked from [blitzstudios/snowflake].

Some code ported from [Sonato/snowflake].

[blitzstudios/snowflake]: https://github.com/blitzstudios/snowflake
[sonato/snowflake]: https://github.com/Sonato/snowflake
[blitz studios, inc.]: https://github.com/blitzstudios
