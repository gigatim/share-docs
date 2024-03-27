---
directory_name: releases_example
layout: doc
toc: true
title: Phoenix deploy with Releases
toc_title: Phoenix with Releases
description: |
  Go from zero to hero with Gigalixir, Phoenix, and Elixir Releases. This guide uses the latest versions of erlang, elixir, node, and phoenix.
order: 010
updated: 2024-02-12
version_elixir: 1.16.1
version_erlang: 26.2.2
version_node: 21.6.1
version_phoenix: 1.7.11
permalink: /docs/getting-started-guide/phoenix-releases-deploy
aliases: 
  - /docs/getting-started-guide/phoenix-releases-deploy.html
---

{% if jekyll.environment == "development" %}
  {% assign directory_name = "gigalixir_getting_started" %}
{% else %}
  {% assign directory_name = page.directory_name %}
{% endif %}

The following instructions allow a user to go from zero to hero with Gigalixir, Phoenix, 
and Elixir Releases.
This guide uses the latest versions:
* [erlang](https://www.erlang.org/) {{ page.version_erlang }}
* [elixir](https://elixir-lang.org/) {{ page.version_elixir }}
* [node](https://nodejs.org/) {{ page.version_node }}
* [phoenix](https://www.phoenixframework.org/) {{ page.version_phoenix }}

Last updated/tested on {{ page.updated }}.


## Prerequisites

This guide utilizes these tools:
* [ASDF tool](https://asdf-vm.com/)
* [Gigalixir CLI](/docs/getting-started-guide#install-the-command-line-interface)
* [Git](https://git-scm.com/)
* [NVM](https://github.com/nvm-sh/nvm)

With these tools set up, let's get started!


## Install Erlang, Elixir, and Node

Use ASDF and NVM to install the versions we want.
{% capture code %}
```
asdf install erlang {{ page.version_erlang }}
asdf install elixir {{ page.version_elixir }}
nvm install v{{ page.version_node }}
```
{% endcapture %}
{% include code-block.html code=code %}

Now set the versions as active.
{% capture code %}
```
asdf local erlang {{ page.version_erlang }}
asdf local elixir {{ page.version_elixir }}
nvm use {{ page.version_node }}
```
{% endcapture %}
{% include code-block.html code=code %}


## Install Phoenix Framework
{% if jekyll.environment == "development" %}
  {% assign mix_force = " --force" %}
{% else %}
  {% assign mix_force = "" %}
{% endif %}

Install hex
{% capture code %}
```
mix local.hex{{ mix_force }}
```
{% endcapture %}
{% include code-block.html code=code %}

Get the latest version of Phoenix and install it.
{% capture code %}
```
mix archive.install hex phx_new{{ mix_force }}
```
{% endcapture %}
{% include code-block.html code=code %}


## Create a new phoenix application

Create the phoenix application, in this case `{{ directory_name }}` is the name of the application.
You will probably want use some other name.

{% capture code %}
```
mix phx.new {{ directory_name }} --install
```
{% endcapture %}
{% include code-block.html code=code %}

Now move into the new project directory and make it a git repository.
{% capture code %}
```
cd {{ directory_name }}
git init
git add .
git commit -m "initial commit: mix phx.new"
```
{% endcapture %}
{% include code-block.html code=code %}


## Set the versions for the buildpacks to use

Gigalixir uses a buildpack system.
We can set the versions in the `elixir_buildpack.config` and `phoenix_static_buildpack.config` to ensure a consistent environment between development and production.

{% capture code %}
```
echo "elixir_version={{ page.version_elixir }}" > elixir_buildpack.config
echo "erlang_version={{ page.version_erlang }}" >> elixir_buildpack.config
echo "node_version={{ page.version_node }}" > phoenix_static_buildpack.config

git add elixir_buildpack.config phoenix_static_buildpack.config
git commit -m "set elixir, erlang, and node version"
```
{% endcapture %}
{% include code-block.html code=code %}


## Esbuild support

Gigalixir has esbuild installed, but we need to tell the buildpacks how to compile our assets.

{% capture code %}
```
mkdir -p assets

echo '{
  "scripts": {
    "deploy": "cd .. && mix assets.deploy && rm -f _build/esbuild"
  }
}' > assets/package.json

git add assets/package.json
git commit -m "compile assets on build"
```
{% endcapture %}
{% include code-block.html code=code %}


## Use Elixir Releases instead of Mix

Newer versions of Elixir use a `runtime.exs` file instead of `releases.exs`.
Gigalixir's buildpacks will want a `releases.exs` to determine this is not a Mix deployment.
This file can be empty, it just needs to exist.

{% capture code %}
```
echo "# this file is here so Gigalixir uses Elixir Releases on deploy" \
  > config/releases.exs
git add config/releases.exs
git commit -m "use elixir releases"
```
{% endcapture %}
{% include code-block.html code=code %}


## Create the gigalixir app and database

The default phoenix application is going to require a DATABASE_URL.
In the below example, we create a [free tier database]().
If you plan to use this application for production, consider the [standard tier database].
{% if jekyll.environment == "development" %}
  {% assign db_create = " --yes" %}
{% else %}
  {% assign db_create = "" %}
{% endif %}

{% capture code %}
```
APP_NAME=$(gigalixir create)
gigalixir pg:create --free{{ db_create }}
```
{% endcapture %}
{% include code-block.html code=code %}

## Setup the web endpoint

The newer versions of phoenix can quickly be deployed by setting the `PHX_HOST` and `PHX_SERVER` environment variables instead of editing the endpoint configuration.
For [custom domains](/docs/domain) you will want to make further modifications.

{% capture code %}
```
gigalixir config:set PHX_HOST=${APP_NAME}.gigalixirapp.com
gigalixir config:set PHX_SERVER=true
```
{% endcapture %}
{% include code-block.html code=code %}


## Push to gigalixir and prosper

We are ready to deploy.

{% capture code %}
```
git push -u gigalixir main
```
{% endcapture %}
{% include code-block.html code=code %}

You can monitor the deployment using `gigalixir ps`.  If you run into issues, check `gigalixir logs`.

As always, if you hit any issues, we're one email away to help. Just write us at [Gigalixir Support](mailto:support@gigalixir.com).


{% if jekyll.environment == "development" %}
## Teardown

To teardown the app and code we just tested with run the following.

{% capture code %}
```
gigalixir ps:scale -r 0

while ! gigalixir ps | jq '.pods | length' | grep -q "0"; do
  sleep 1
done
DATABASE_ID=$(gigalixir pg | jq -r '.[].id')
gigalixir pg:destroy -d $DATABASE_ID -y

gigalixir apps:destroy -y

cd ..
rm -rf {{ directory_name }}
```
{% endcapture %}
{% include code-block.html code=code %}
{% endif %}
