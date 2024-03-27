---
layout: doc
toc: true
title: Configuration
description: Setting up Gigalixir App configurations through environment
  variables. Get, set, and delete configs. Note that setting configs
  automatically restarts your app.
order: 70
permalink: /docs/config
aliases:
  - /docs/config.html
---
## How to Configure an App

All app configuration is done through environment variables. You can get, set, and delete configs using the following commands. 

::: note
::: title
Note
:::
Setting configs *automatically* restarts your app.
:::

```bash
gigalixir config
gigalixir config:set FOO=bar
gigalixir config:unset FOO
```

## Environment Variables and Secrets

Environment variables in general are confusing because Mix, Distillery, and Elixir Releases all handle it differently.

For Distillery, [Distillery's Runtime Configuration](https://hexdocs.pm/distillery/config/runtime.html){:target="_blank"} explains it well, but in short:

For Distillery, ***never*** use `System.get_env("FOO")` in your `prod.exs`. Always use `"${FOO}"` instead. 

For Mix, always use `System.get_env("FOO")` in your `prod.exs`.

For Elixir Releases, always use `System.get_env("FOO")`. Make sure to put it in your `releases.exs` file to load at runtime, (*which is usually preferable*).

For example with Distillery, to introduce a new `MY_CONFIG` env var, add to your `prod.exs` file the following:

```elixir
config :myapp,
    my_config: "${MY_CONFIG}"
```

Then set the `MY_CONFIG` environment
variable, by running

```bash
gigalixir config:set MY_CONFIG=foo
```

In your app code, access the environment variable using

```elixir
Application.get_env(:myapp, :my_config) == "foo"
```

::: note
::: title
Note
:::

Elixir 1.11 adds `config/runtime.exs`. If you use that instead, then you'll want to specify buildpacks since we can no longer detect if you want releases or mix mode. See [How to Specify Buildpacks (*optional*)](/docs/modify-app/releases#specify-buildpacks-optional).
:::

## How to Copy Configuration Variables

```bash
gigalixir config:copy -s $SOURCE_APP -d $DESTINATION_APP
```

Note, this will copy all configuration variables from the source to the
destination. If there are duplicate keys, the destination config will be
overwritten. Variables that only exist on the destination app will not
be deleted.

## How to Use a Custom vm.args

Gigalixir generates a default `vm.args` file for you and tells Distillery to use it by setting the `VMARGS_PATH` environment variable. By default, it is set to `/release-config/vm.args`. If you want{.interpreted-text role="elixir"} to use a custom `vm.args`, we recommend you follow these instructions.

Disable Gigalixir's default vm.args

```bash
gigalixir config:set GIGALIXIR_DEFAULT_VMARGS=false
```

Create a `rel/vm.args` file in your repository. As an example, we provide our [gigalixir-getting-started's vm.args file](https://github.com/gigalixir/gigalixir-getting-started/blob/js/distillery-2.0/rel/vm.args){:target="_blank"}.

Lastly, you need to modify your distillery config so it knows where to find your `vm.args` file, like the below sample. 

*For a more complete example, see [gigalixir-getting-started's rel/config.exs file](https://github.com/gigalixir/gigalixir-getting-started/blob/js/distillery-2.0/rel/config.exs#L41){:target="_blank"}*.

```elixir
...
environment :prod do
  ...
  # this is just to get rid of the warning. see https://github.com/bitwalker/distillery/issues/140
  set cookie: :"${MY_COOKIE}"
  set vm_args: "rel/vm.args"
end
...
```

After a new deploy, verify by SSH'ing into your instance and inspecting your release's vm.arg file:

```bash
gigalixir ps:ssh
cat /app/var/vm.args
```

## How to Specify which Release, Environment, or Profile to Build for Distillery

If you have multiple releases defined in `rel/config.exs`, which is common for umbrella apps, you can specify which release to build by setting a config variable on your app that controls the options passed to *mix distillery.release*. 

For example, you can pass the *\--profile* option using the command below.

```bash
gigalixir config:set GIGALIXIR_RELEASE_OPTIONS="--profile=$RELEASE_NAME:$RELEASE_ENVIRONMENT"
```

With this config variable set on each of your Gigalixir apps, when you deploy the same repo to each app, you'll get a different release.

If you have multiple Phoenix apps in the umbrella, instead of deploying
each as a separate distillery release, you could also consider something
like this [master_proxy](https://github.com/jesseshieh/master_proxy) to
proxy requests to the two apps.

## How to Specify which Release, Environment, or Profile to Build for Elixir Releases

If you want to pass options to `mix release` such as the release name, you can specify options with the `GIGALIXIR_RELEASE_OPTIONS` env var.

For example, to build a different release other than the default, run:

```bash
gigalixir config:set GIGALIXIR_RELEASE_OPTIONS="my-release"
```

## How Do I Use a Private Hex Dependency?

First, take a look at the following page and generate an auth key for
your org: 

<https://hex.pm/docs/private#authenticating-on-ci-and-build-servers>

Then add to your `elixir_buildpack.config` file:

```bash
hook_pre_fetch_dependencies="mix hex.organization auth myorg --key ${HEX_ORG_AUTH}"
```

Finally, run:

```bash
gigalixir config:set HEX_ORG_AUTH="authkeyhere"
```

## How do I use Webpack, Yarn, Bower, Gulp (*etc.*) Instead of Brunch?

Simply, you can use a custom compile script. 

For more details, see
<https://github.com/gigalixir/gigalixir-buildpack-phoenix-static#compile>{:target="_blank"}

Here is an example script that we've used for webpack:

```bash
cd $assets_dir
node_modules/.bin/webpack -p

cd $phoenix_dir
mix "${phoenix_ex}.digest"
```

## How do I Specify my Elixir, Erlang, Node, NPM, (*etc*) Versions?

Your Elixir and Erlang versions are handled by the heroku-buildpack-elixir buildpack. To configure, see the [heroku-buildpack-elixir
configuration](https://github.com/HashNuke/heroku-buildpack-elixir#configuration){:target="_blank"}.

In short, you specify them in a `elixir_buildpack.config` file.

Node and NPM versions are handled by the gigalixir-buildpack-phoenix-static buildpack. To configure, see the [gigalixir-buildpack-phoenix-static configuration](https://github.com/gigalixir/gigalixir-buildpack-phoenix-static#configuration){:target="_blank"}.

In short, you specify them in a `phoenix_static_buildpack.config` file.

Supported Elixir and Erlang versions can be found at
<https://github.com/HashNuke/heroku-buildpack-elixir#version-support>{:target="_blank"}

## How do I Specify which Buildpacks I Want to Use?

Normally, the buildpack you need is auto-detected for you, but in some cases, you may want to specify which buildpacks you want to use. 

To do this, create a `.buildpacks` file and list each buildpack you want to use. For example, the default buildpacks for Elixir apps using Distillery would look like:

```bash
https://github.com/HashNuke/heroku-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-distillery.git
```

The default buildpacks for Elixir apps running Mix looks like:

```bash
https://github.com/HashNuke/heroku-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-mix.git
```

Note the last buildpack. It's there to make sure your `Procfile` is set up correctly to run on Gigalixir. It basically makes sure you have your node name and cookie
set correctly so that Remote Console, migrate, observer, etc will work.

## Can I use a Custom Procfile?

Definitely! 

If you are using Mix (not Distillery) and you have a `Procfile` at the root of your repo, we'll use it instead of the default one.

If you are using Distillery, you'll have to use distillery overlays to include the Procfile inside your release tarball *i.e. slug*. 

If you are using Elixir releases, then you want to place the Procfile inside rel/overlays so that it gets copied into the release tarball.

The only issue is that if you want Remote Console to work, you'll want to make sure the node name and cookie are set properly. For example, your `Procfile` should be set as follows:

```bash
web: elixir --name $MY_NODE_NAME --cookie $MY_COOKIE -S mix phoenix.server
```

## Can I Choose my Operating System, Stack, or Image?

We have two stacks you can choose from: `gigalixir-18`, and `gigalixir-20`.

These stacks are based on Heroku's heroku-18 and heroku-20,
respectively which are based on Ubuntu 18 and 20 respectively.

Please note that `gigalixir-20` is the default.

Note that some older apps on gigalixir might be running gigalixir-14 or
gigalixir-16, based on Heroku's cedar-14 and heroku-16, which will be
end-of-life on November 2nd, 2020 and May 1st, 2021. 

gigalixir-14 and gigalixir-16 will be also be end-of-life on the same day. 

See <https://devcenter.heroku.com/changelog-items/1757>{:target="_blank"} and <https://help.heroku.com/0S5P41DC/heroku-16-end-of-life-faq>{:target="_blank"}

You can choose your stack when you create your app with

```bash
gigalixir create --stack gigalixir-20
```

or you can change it later on with

```bash
# Note that depending on the situation, you may have to re-deploy your app after changing the stack in case the shared libraries have changed locations.
gigalixir stack:set --stack gigalixir-20
```

You can see what stack you are running with `gigalixir apps:info` or `gigalixir ps`.

For information about what packages are available in each stack, see
<https://devcenter.heroku.com/articles/stack-packages>{:target="_blank"}, as well as, the Dockerfiles at <https://github.com/gigalixir/gigalixir-run>{:target="_blank"}

## Can I Run my App in AWS instead of Google Cloud Platform? What about Europe?

Yes, if your current infrastructure is on AWS, you'll probably want to run your Gigalixir app on AWS too. Or if most of your users are in Europe, you probably want to host your app in Europe. 

We currently support GCP us-central1 and GCP europe-west1 as well as AWS us-east-1 and AWS us-west-2. 

When creating your app with `gigalixir create` simply specify the `--cloud=aws` and `--region=us-east-1` options.

Once the app is created, it's difficult to migrate to another region. 

If you want to migrate after an app is created, Heroku's guide is a good overview of what you should consider. 

If you don't mind downtime, the transition could be easy, but unfortunately Gigalixir isn't able to do it for you as a one-click button press. See <https://devcenter.heroku.com/articles/app-migration>{:target="_blank"}

One thing to keep in mind is that Gigalixir Postgres databases are as of right now only available in GCP/us-central1 and GCP/europe-west1, however, we can set up a database for you in AWS manually if you like.

Just [contact us](mailto:support@gigalixir.com){:target="_blank"} and we'll create one for you. We plan to add AWS to the Gigalixir CLI soon.

If you don't see the region you want, please [contact us](mailto:support@gigalixir.com){:target="_blank"} and let us know, as we open new regions based purely on demand.

## What Built-in Environment Variables are Available to my App?

**SOURCE_VERSION**: Contains the current SHA.

**HOST_INDEX**: contains the index of the replica. 

The hostname for each replica is randomly generated which can be a problem for services like
DataDog and NewRelic who charge by the host. 

We also keep a sort of ordered list of your replicas that you can use to report hostnames to keep your number of hosts low. 

Each replica currently running will have a different HOST_INDEX, but once a replica is terminated, its HOST_INDEX can be re-used in another replica.

**APP_NAME**: The Gigalixir app name.

**APP_KEY**: The app specific key required to fetch information about your app from inside the replica. 

You probably *don't* need to use this variable - unless you're doing something really low level - but it's there if you need it.

**ERLANG_COOKIE**: Contains a randomly generated UUID that we use as your erlang distribution cookie. 

We set it for you automatically and it's used in your default `vm.args` file so you don't need to mess with anything, but this is how to find it if you should want to use it for something else.

**LOGPLEX_TOKEN**: Contains the app specific token we use to send your app logs to logplex. 

Logplex is our central log router which handles aggregating, draining, and tailing your logs. 

You can use this if you want to do something custom with logs that can't be done by printing to `stdout` from your app.

**MY_POD_IP**: Contains your replica/container/pod's IP address.

**PORT**: Contains the port your app needs to listen on to pass health checks and receive traffic. 

It is almost always 4000, but we reserve the right to change or randomize it.

**SECRET_KEY_BASE**: Contains a randomly generated string that we use as your Elixir app's secret key base.

**HOME**: Contains the location of your app's home directly. 

It is almost always /app, but we reserve the right to change it, so we list this just to be sure.

## How do I get a Static Outgoing IP address?

Gigalixir supports static outgoing IP addresses for customers using [QuotaGuard Static IP's](/blog/new-owners-gigalixir/) - which are affordable and simple to setup and configure. 

Just configure your http client to make requests through the QuotaGuard Static IP outbound proxy. 

For example, with HTTPoison, something like this works quickly and easily:

```elixir
HTTPoison.get(url, [], [proxy: {:socks5, String.to_charlist("server_domain"), port_num}, socks5_user: "username", socks5_pass: "password"])
```
