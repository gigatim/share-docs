---
layout: doc
toc: true
title: Database Management
description: How to provision, upgrade/migrate, scale, dump, restore, restart,
  delete, install and connect to a PostgreSQL or CloudSQL database, run
  migrations and run seeds
order: 60
permalink: /docs/database
aliases:
  - /docs/database.html
---
## Database Tiers

{: #tiers }

Gigalixir offers two tiers of databases: free and standard.

**Free Tier** databases are not suitable for production use.
We provide them to users to help them get their development started.

Free tier databases run on a multi-tenant postgres database cluster with shared CPU, memory, and disk.
You are limited to two connections, 10,000 rows, and no automatic backups.
Idle connections are terminated after five minutes.  

[Free tier databases](/docs/database/free-tier) attached to applications that are inactive for an extended period of time will be destroyed.

**Standard Tier** database are production quality databases.
High availability and [read replicas](/blog/read-replica-databases/) are available.
Backups are automatic and daily.
Connection limits are much higher.
Disk size scales with your usage.

See [Database Tier Comparison](#tier-comparison) for more details.

Continue reading for general information applicable to both tiers.\
For more information on creating, managing, and operating databases, see:

* [Standard Tier Databases](/docs/database/standard-tier)

  * [Read Replicas](/docs/database/read-replicas)
  * [Backup and Recovery](/docs/database/backup-and-recovery)
* [Free Tier Databases](/docs/database/free-tier)

  * [Upgrading to Standard Database](/docs/database/free-tier#upgrading)

## Connecting to a Database

{: #connecting }

If you followed the [Getting Started Guide](/docs/getting-started-guide),
then your database should already be connected.
If not, connecting to a database is done no differently from apps running outside Gigalixir. 

We recommend you set a DATABASE_URL config and configure your database adapter accordingly.
In short, you'll want to add something like this to your `prod exs` file.

```elixir
config :gigalixir_getting_started, GigalixirGettingStarted.Repo,
  adapter: Ecto.Adapters.Postgres,
  url: {:system, "DATABASE_URL"},
  database: "",
  ssl: true,
  pool_size: 2
```

Replace `:gigalixir_getting_started` and `GigalixirGettingStarted` with your app name.
Then, be sure to set your `DATABASE_URL` config. 

For more information on setting configs, see [How to Configure an App](/docs/config#how-to-configure-an-app).
If you provisioned your database using the Gigalixir Console or CLI,
then `DATABASE_URL` should be set for you automatically once the database is provisioned. 

Otherwise, do the following: 

```bash
gigalixir config:set DATABASE_URL="ecto://{user}:{pass}@{host}:{port}/{db}"
```

## Runnning Migrations

{: #running-migrations }

We try to make it easy by providing a special command.
The command runs on your existing app container,
so you'll need to make sure your app is running first and set up your SSH keys.

```bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"
```

Then run:

```bash
gigalixir ps:migrate
```

This command runs your migrations in a Remote Console directly on your production node.
It makes some assumptions about your project so if it does not work, please 
[contact us for help](/docs/troubleshooting#supporthelp).

### Umbrella App Migrations

{: #running-migrations-with-an-umbrella }

If you are running an Umbrella app,
you will probably need to specify which "inner app" within your Umbrella to migrate.
Do this by passing the `--migration_app_name` flag like so

```bash
gigalixir ps:migrate --migration_app_name=$MIGRATION_APP_NAME
```

When running `gigalixir ps:migrate`, sometimes the migration doesn't do exactly what you want.
If you need to tweak the migration command to fit your situation,
it helps to know that all `gigalixir ps:migrate` is doing is dropping into a remote_console and running the following:

```elixir
repo = List.first(Application.get_env(:gigalixir_getting_started, :ecto_repos))
app_dir = Application.app_dir(:gigalixir_getting_started, "priv/repo/migrations")
Ecto.Migrator.run(repo, app_dir, :up, all: true)
```

*For information on how to open a Remote Console, see
[How to Drop into a Remote Console](/docs/runtime#how-to-drop-into-a-remote-console)*.

For example if you have more than one app,
you may not want to use *List.first* to find the app that contains the migrations.

### Chicken Egg Migration Problems

{: #chicken-egg }

If you have a chicken-and-egg problem where your app will not start without migrations run,
and migrations won't run without an app running,
you can try the following workaround on your local development machine.
This will run migrations on your production database from your local machine using your local code.

```bash
MIX_ENV=prod DATABASE_URL="$YOUR_PRODUCTION_DATABASE_URL" mix ecto.migrate
```

## Running Migrations at Startup

{: #running-migrations-at-startup }

::: important
::: title
Warning
:::
Application startup has a 10 minute timeout.
If your migration is rather large, then we recommend not using this technique.
Consider the manual migrations listed above or running the migration from a local workstation by setting
MIX_MODE to prod and setting your DATABASE_URL similar to the [Chicken Egg solution](#chicken-egg).
:::

### Releases Migrations at Start

{: #running-migrations-at-startup-with-releases }
If you are using Elixir Releases, we suggest creating a custom Procfile and overlaying it into your release tarball. 

To do this create a file `rel/overlays/Procfile` with something like this:

```bash
web: /app/bin/$GIGALIXIR_APP_NAME eval "MyApp.Release.migrate" && /app/bin/$GIGALIXIR_APP_NAME $GIGALIXIR_COMMAND
```

You have to implement the `MyApp.Release.migrate` function with something like
<https://hexdocs.pm/phoenix/releases.html#ecto-migrations-and-custom-commands>{:target="_blank"}.

You might also be interested in reading
<https://elixirforum.com/t/equivalent-to-distillerys-boot-hooks-in-mix-release-elixir-1-9/23431>{:target="_blank"}

### Mix Migrations at Start

{: #running-migrations-at-startup-with-mix }

If you aren't running Elixir Releases or Distillery, meaning you are in Mix mode, you can try modifying your `Procfile` to something like this

```bash
web: mix ecto.migrate && elixir --name $MY_NODE_NAME --cookie $MY_COOKIE -S mix phx.server
```

For more details, see [Can I Use a Custom Procfile](/docs/config#can-i-use-a-custom-procfile)?

### Distillery Migrations at Start

{: #running-migrations-at-startup-with-distillery }

If you are using Distillery, we suggest using a Distillery pre-start boot hook by following
<https://github.com/bitwalker/distillery/blob/master/docs/guides/running_migrations.md>{:target="_blank"}
and
<https://github.com/bitwalker/distillery/blob/master/docs/extensibility/boot_hooks.md>{:target="_blank"}

## Reset a Database

{: #resetting }

[Drop into a Remote Console](/docs/runtime#how-to-drop-into-a-remote-console) and run this to "down" migrate.
You may have to tweak the command depending on what your app is named and if you're running an umbrella app.

```elixir
Ecto.Migrator.run(MyApp.Repo, Application.app_dir(:my_app, "priv/repo/migrations"), :down, [all: true])
```

Then run this to "up" migrate.

```elixir
Ecto.Migrator.run(MyApp.Repo, Application.app_dir(:my_app, "priv/repo/migrations"), :up, [all: true])
```

## Seeding a Database

{: #seeding }

If you are in Mix mode (*not using Distillery or Elixir releases*) and
have a seeds.exs file, you can just run

```bash
gigalixir run -- mix run priv/repo/seeds.exs
```

Otherwise, you'll need to [drop into a Remote Console](/docs/runtime#how-to-drop-into-a-remote-console) and run commands manually. If you have a `seeds.exs` file, you can follow [the Distillery Migration Guide](https://hexdocs.pm/distillery/guides/running_migrations.html){:target="_blank"} and run something like this in your [Remote Console.](/docs/runtime)
```elixir
seed_script = Path.join(["#{:code.priv_dir(:myapp)}", "repo", "seeds.exs"])
Code.eval_file(seed_script)
```

## Manual Backups

{: #manual-backups }

For point in time backup or migration of data, we recommend `pg_dump`. 

You can find all the connection parameters you need from `gigalixir pg`. 

This should dump the database contents as a sql file which you can load back in with `psql`. 

If you dump a binary file, then you can use `pg_restore`.

## Tier Comparison

{: #tier-comparison }

{:.narrow-table}
| Database Feature          | FREE Tier | STANDARD Tier |
| ------------------------- | :-------: | :-----------: |
|  SSL Connections          |   YES     |    YES        |
|  Data Import/Export       |   YES     |    YES        |
|  Data Encryption          |           |    YES        |
|  Dedicated CPU            |           |    YES\[1]   |
|  Dedicated Memory         |           |    YES        |
|  Dedicated Disk           |           |    YES        |
|  No Connection            |           |    YES        |
|  High Availability        |           |    YES\[2]   |
|  Read Replicas Available  |           |    YES        |
|  Limits No                |           |    YES        |
|  Row Limits               |           |    YES        |
|  Backups                  |           |    YES        |
|  Scalable/Upgradeable     |           |    YES        |
|  Automatic Data Migration |           |    YES        |
|  Extensions               |           |    YES        |
|  Functions                |           |    YES        |
|  Triggers                 |           |    YES        |
|  Role Management          |           |    YES        |

* \[1] Standard databases with size 4 and above have dedicated CPU.
* \[2] Standard databases default to single instance, but can have high availability enabled.

<script>
window.onload = function() {
  if (window.location.hash === "#how-to-provision-a-free-postgresql-database") {
    window.location.href = "/docs/database/free-tier#creating";

  } else if (window.location.hash === "#how-to-upgrade--migrate-a-free-db-to-a-standard-db") {
    window.location.href = "/docs/database/free-tier#upgrading";

  } else if (window.location.hash === "#how-to-provision-a-standard-postgresql-database") {
    window.location.href = "/docs/database/standard-tier#creating";

  } else if (window.location.hash === "#how-to-scale-a-database") {
    window.location.href = "/docs/database/standard-tier#scaling";

  } else if (window.location.hash === "#how-to-restart-a-database") {
    window.location.href = "/docs/database/standard-tier#restarting";

  } else if (window.location.hash === "#how-to-restore-a-database-backup") {
    window.location.href = "/docs/database/backup-and-recovery";

  } else if (window.location.hash === "#how-to-delete-a-database") {
    window.location.href = "/docs/database/standard-tier#destroying";

  } else if (window.location.hash === "#how-to-install-a-postgres-extension") {
    window.location.href = "/docs/database/standard-tier#postgres-extensions";

  } else if (window.location.hash === "#how-to-connect-a-database") {
    window.location.href = "#connecting";

  } else if (window.location.hash === "#how-to-run-migrations") {
    window.location.href = "#running-migrations";

  } else if (window.location.hash === "#how-to-run-migrations-on-startup") {
    window.location.href = "#running-migrations-at-startup";

  } else if (window.location.hash === "#how-to-reset-the-database") {
    window.location.href = "#resetting";

  } else if (window.location.hash === "#how-to-run-seeds") {
    window.location.href = "#seeding";

  } else if (window.location.hash === "#how-to-dump-the-database-to-a-file") {
    window.location.href = "#manual-backups";
  }
};
</script>
