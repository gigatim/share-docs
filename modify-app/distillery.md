---
layout: doc
toc: true
title: Modifying an App for Gigalixir using Distillery
toc_title: Distillery
description: To modify your app to use Distillery on Gigalixir to build releases, follow these simple instructions and example app. 
order: 033
permalink: /docs/modify-app/distillery
aliases: 
  - /docs/modify-app/distillery.html
---

{% include distillery-deprecation.md %}

_For an example app that uses Distillery and works on Gigalixir, please see
[https://github.com/gigalixir/gigalixir-getting-started](https://github.com/gigalixir/gigalixir-getting-started){:target="\_blank"}_

## Install Distillery to Build Releases

To install Distillery to build releases, you'll need to add something similar to this to the
`deps` list in `mix.exs`

``` elixir
{:distillery, "~> 2.1"}
```

::: important
::: title
Important
:::
If you are running Elixir 1.9, then you _must_ use Distillery 2.1 or greater. Elixir 1.9 and Distillery below 2.1 both use _**mix release**_ and Elixir's always takes precedence. Distillery 2.1 renames the task to _**mix distillery.release**_.
:::

Then, run: 

``` bash
mix deps.get
mix distillery.init
```
(_If you are running distillery below version 2.1, you'll want to run `mix release.init` instead._)

Don't forget to commit:

``` bash
git add mix.exs mix.lock rel/
git commit -m 'install distillery'
```

## Configuration and Secrets

Add something like the following in `prod.exs`

``` elixir
config :gigalixir_getting_started, GigalixirGettingStartedWeb.Endpoint,
  server: true, # Without this line, your app will not start the web server!
  load_from_system_env: true, # Needed for Phoenix 1.3. Doesn't hurt for other versions
  http: [port: {:system, "PORT"}], # Needed for Phoenix 1.2 and 1.4. Doesn't hurt for 1.3.
  secret_key_base: "${SECRET_KEY_BASE}",
  url: [host: "${APP_NAME}.gigalixirapp.com", port: 443],
  cache_static_manifest: "priv/static/cache_manifest.json",
  version: Mix.Project.config[:version] # To bust cache during hot upgrades

config :gigalixir_getting_started, GigalixirGettingStarted.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: "${DATABASE_URL}",
  database: "", # Works around a bug in older versions of ecto. Doesn't hurt for other versions.
  ssl: true,
  pool_size: 2 # Free tier db only allows 4 connections. Rolling deploys need pool_size*(n+1) connections where n is the number of app replicas.
```

::: attention
::: title
Attention
:::
`server: true` **is very important and is commonly left out. Make sure you have this line included.**
:::

1. Replace `:gigalixir_getting_started` with your app name e.g. `:my_app`
2. Replace `GigalixirGettingStartedWeb.Endpoint` with your endpoint module name. You can find your endpoint module name by running something like:

    ``` bash
    grep -R "defmodule.*Endpoint" lib/
    ```
    Phoenix 1.2, 1.3, and 1.4 give different names, **so this is a common
    source of errors**.

3. Replace `GigalixirGettingStarted.Repo` with your repo module name e.g. `MyApp.Repo`

You don't have to worry about setting your `SECRET_KEY_BASE` config because we generate one and set it for you. However, if you don't use a Gigalixir managed postgres database, you'll have to set the `DATABASE_URL` yourself.

## Verify

Let's make sure everything works.

First, try building static assets:

``` bash
mix deps.get
cd assets
npm install
npm run deploy
cd ..
mix phx.digest
```

and building a Distillery release locally:

``` bash
SECRET_KEY_BASE="$(mix phx.gen.secret)" MIX_ENV=prod DATABASE_URL="postgresql://user:pass@localhost:5432/foo" mix distillery.release --env=prod
# if you are running distillery below 2.1, you'll want to run this instead: MIX_ENV=prod mix release --env=prod
```

and running it locally:

``` bash
MIX_ENV=prod APP_NAME=gigalixir_getting_started SECRET_KEY_BASE="$(mix phx.gen.secret)" DATABASE_URL="postgresql://user:pass@localhost:5432/foo" MY_HOSTNAME=example.com MY_COOKIE=secret REPLACE_OS_VARS=true MY_NODE_NAME=foo@127.0.0.1 PORT=4000 _build/prod/rel/gigalixir_getting_started/bin/gigalixir_getting_started foreground
```

Don't forget to replace `gigalixir_getting_started` with your own app name. Also, change/add the environment variables as needed.

Commit the changes:

``` bash
git add config/prod.exs assets/package-lock.json
git commit -m 'distillery configuration'
```

Check it out to see if it works:

``` bash
curl localhost:4000
```

If that didn't work, the first place to check is
`prod.exs`. Make sure you have
`server: true` somewhere and there are
no typos.

Also check out [troubleshooting](/docs/troubleshooting#troubleshooting-page).

If it still doesn't work, don't hesitate to
[contact our Support](/docs/troubleshooting#supporthelp).

If everything works, continue on to [Set Up App for Deploys](/docs/getting-started-guide#set-up-app-for-deploys).

## Specify Buildpacks (_optional_)

We rely on buildpacks to compile and build your release. 

We auto-detect a variety of buildpacks so you probably don't need this, however, if you want to specify your own buildpacks, then create a `.buildpacks` file with the buildpacks
you want. 

For example,

``` bash
https://github.com/HashNuke/heroku-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-distillery.git
```

`gigalixir-buildpack-phoenix-static` is optional if you do not have Phoenix static assets. 

For more information about buildpacks, see the [Life of a Deploy](/docs/concepts#life-of-a-deploy).

Note, that the command that gets run in production depends on your
last buildpack.

- If the last buildpack is `gigalixir-buildpack-distillery`, then the command run will be `/app/bin/foo foreground`.
- If the last buildpack is `gigalixir-buildpack-phoenix-static`, then the command run will be `mix phx.server`.
- If the last buildpack is `heroku-buildpack-elixir`, then the command run will be `mix run --no-halt`.

If your command is `mix run --no-halt`, but you are running Phoenix (_just not the assets pipeline_), make sure you set `server: true` in `prod.exs`.

## Set Up Node Clustering with Libcluster (_optional_)

If you want to cluster nodes, you should install libcluster. 

For more information about installing libcluster, see [Clustering your Nodes](/docs/cluster#clustering-nodes).

## Set Up Hot Upgrades with Git v2.9.0

To run hot upgrades, you send an extra http header when running
`git push gigalixir`. 

Extra HTTP headers are only supported in git 2.9.0 and above so make sure you upgrade if
needed. 

For information on how to install the latest version of git on Ubuntu, see [this StackOverflow question](http://stackoverflow.com/questions/19109542/installing-latest-version-of-git-in-ubuntu).

For information on running hot upgrades, see [Hot Upgrade](/docs/deploy#how-to-hot-upgrade-an-app) and [Life of a Hot Upgrade](/docs/concepts#life-of-a-hot-upgrade).
