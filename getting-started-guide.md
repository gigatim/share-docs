---
layout: doc
toc: true
title: Getting Started Guide
description: >
  The goal of this guide is to get your app up and running on Gigalixir. You
  will sign up for an account, prepare your app, deploy, and provision a
  database.
order: 20
permalink: /docs/getting-started-guide/
aliases:
  - /docs/getting-started-guide.html
---
The goal of this guide is to get your app up and running on Gigalixir.
You will sign up for an account, prepare your app, deploy, and provision
a database.

If you're deploying an open source project, we provide consulting
services free of charge. [Contact us](/docs/troubleshooting#help) and we'll send you a pull request with everything you need
to get started.

## Prerequisites

::: tabs
::: group-tab
macOS

1. `python3`.
2. `pip3`. For help, take a look at the
   [pip documentation](https://packaging.python.org/installing/){:target="_blank"}.
3. `git`. For help, take a look at the
       [git
       documentation](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git){:target="_blank"}.
   :::
   ::: group-tab
   Linux
4. `python3`.
5. `pip3`. For help, take a look at the
   [pip documentation](https://packaging.python.org/installing/){:target="_blank"}{:target="_blank"}.
6. `git`. For help, take a look at the
   [git
   documentation](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git){:target="_blank"}.

For example, run

```bash
sudo apt-get update
sudo apt-get install -y python3 python3-pip git-core curl
```

:::
::: group-tab
Windows

1. `python3`.
2. `pip3`. For help, take a look at the
   [pip documentation](https://packaging.python.org/installing/){:target="_blank"}.
3. `git`. For help, take a look at the
       [git
       documentation](https://git-scm.com/book/en/v2/Getting-Started-Installing-Git){:target="_blank"}.
   :::
   :::

## Install the Command-Line Interface

Next, install the command-line interface. Gigalixir has a web interface
at <https://console.gigalixir.com/>{:target="_blank"}, but you will likely still want the
CLI.

::: tabs
::: group-tab
macOS

```bash
pip3 install gigalixir --user
```

Make sure the executable is in your path, if it isn't already.

```bash
echo "export PATH=\$PATH:$(python3 -m site --user-base)/bin" >> ~/.profile
source ~/.profile
```

If you have any trouble, please [reach out for help](/docs/troubleshooting#help){:target="_blank"}.
:::
::: group-tab
Linux

```bash
pip3 install gigalixir --user
```

Make sure the executable is in your path, if it isn't already.

```bash
echo "export PATH=\$PATH:$(python3 -m site --user-base)/bin" >> ~/.profile
source ~/.bash_profile
```

If you have any trouble, please [reach out for help](/docs/troubleshooting#help){:target="_blank"}.
:::
::: group-tab
Windows

```bash
pip3 install gigalixir --user
```

Make sure the executable is in your path, if it isn't already.

On Windows Powershell, try something similar to this. Note this may vary
based on your python version.

```bash
[Environment]::SetEnvironmentVariable("Path", $env:Path + ";$HOME\appdata\roaming\python\python38\Scripts", "Machine")
```

If you have any trouble, please [reach out for help](/docs/troubleshooting#help){:target="_blank"}.

:::
:::

Next, you can verify by running:

```bash
gigalixir --help
```

## Create an Account

If you already have an account, skip this step.

Create an account using the following command. It will prompt you for your email address and password. You will have to confirm your email before continuing. Gigalixir's free tier does not require a credit card, but you will be limited to 1 instance with 0.2GB of memory and 1 postgresql database limited to 10,000 rows.

```bash
gigalixir signup
```

Or if you want to use your Google account:

```bash
gigalixir signup:google
```

## Log In

If you signed up with your Google account, you should already be logged in. Otherwise, log in. This will grant you an API key. It will also optionally modify your ~/.netrc file so that all future commands are authenticated.

```bash
gigalixir login
```

Verify by running:

```bash
gigalixir account
```

## Prepare Your App

Most likely, there is nothing you need to do here and you can skip this
step and "just deploy", but it depends on what version of phoenix
you're running and whether you are okay running in mix mode without
distillery or [elixir releases](https://www.gigalixir.com/docs/modify-app/releases).

For more information, see [modifying an existing app](/docs/modify-app).

Or if you just want to give Gigalixir a spin, clone our reference app.

```bash
git clone https://github.com/gigalixir/gigalixir-getting-started.git
```

Then just follow the instructions in the README under the Deploying section.

## Set Up App for Deploys

To create your app, run the following command. 

It will also set up a git remote. This must be run from within a git repository folder. An app name will be generated for you, but you can also optionally supply an
app name if you wish using
`gigalixir create -n $APP_NAME`. (*There is currently no way to change your app name once it is created*).

 If you like, you can also choose which cloud provider and region using the
`--cloud` and `--region` options. We currently support `gcp` in
`v2018-us-central1` or `europe-west1` and `aws` in `us-east-1` or `us-west-2`. The default
is v2018-us-central1 on gcp.

```bash
cd gigalixir-getting-started
APP_NAME=$(gigalixir create)
```

Verify that the app was created, by running:

```bash
gigalixir apps
```

Verify that a git remote was created by running:

```bash
git remote -v
```

If someone in your organization has already created the Gigalixir app and you only need to add the proper git remote to your local repository configuration, you can skip the app creation and add a the `gigalixir` git remote by using the `git:remote` command:

```bash
gigalixir git:remote $APP_NAME
```

## Specify Versions

The default Elixir version is defined
[here](https://github.com/HashNuke/heroku-buildpack-elixir/blob/master/elixir_buildpack.config){:target="_blank"} which is quite old and it's a good idea to use the same version in
production as you use in development; so let's specify them. 

Supported Elixir and erlang versions can be found at <https://github.com/HashNuke/heroku-buildpack-elixir#version-support>{:target="_blank"}

```bash
echo "elixir_version=1.11.3" > elixir_buildpack.config
echo "erlang_version=23.2" >> elixir_buildpack.config
```

Same for nodejs:

```bash
echo "node_version=14.15.4" > phoenix_static_buildpack.config
```

Don't forget to commit:

```bash
git add elixir_buildpack.config phoenix_static_buildpack.config
git commit -m "set elixir, erlang, and node version"
```

Phoenix v1.6 or greater uses `esbuild` to compile your assets but Gigalixir images come with npm, so we will configure npm directly to deploy our assets. 

Add a `assets/package.json` file if you don't have any with the following:

```bash
{
  "scripts": {
    "deploy": "cd .. && mix assets.deploy && rm -f _build/esbuild"
  }
}
```

Don't forget to commit:

```bash
git add assets/package.json
git commit -m "assets deploy script"
```

## Provision a Database

Phoenix 1.4+ enforces the DATABASE_URL env var at compile time so let's
create a database first, before deploying.

```bash
gigalixir pg:create --free
```

Verify by running:

```bash
gigalixir pg
```

Once the database is created, verify your configuration includes a
`DATABASE_URL` by running:

```bash
gigalixir config
```

## Deploy!

Finally, build and deploy.

```bash
git push gigalixir
```

Wait a minute or two for the app to pass health checks. You can check
the status by running:

```bash
gigalixir ps
```

Once it's healthy, verify it works:

```bash
curl https://$APP_NAME.gigalixirapp.com/
```

Or you could also run: 

```
gigalixir open
```

## Run Migrations

We try to make it easy by providing a special command.
The command runs on your existing app container,
so you'll need to make sure your app is running first and set up your SSH keys.
``` bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"
```

Then run:
``` bash
gigalixir ps:migrate
```

For more, see [How to Run Migrations.](/docs/database#how-to-run-migrations).


## What's Next?

* [How to Configure an App](/docs/config#how-to-configure-an-app)
* [How to Check App Status](/docs/app#how-to-check-app-status)
* [How to Tail Logs](/docs/log#how-to-tail-logs)
* [How to Scale an App](/docs/app#how-to-scale-an-app)
* [How to Restart an App](/docs/app#how-to-restart-an-app)
* [How to Rollback an App](/docs/deploy#how-to-rollback-an-app)
* [How to Drop into a Remote Console](/docs/runtime#how-to-drop-into-a-remote-console)
* [How to Launch a Remote Observer](/docs/runtime#how-to-launch-a-remote-observer)
* [Hot Upgrades](/docs/deploy#how-to-hot-upgrade-an-app)