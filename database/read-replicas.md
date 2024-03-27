---
layout: doc
toc: true
title: Read Replicas for Standard Tier Databases
toc_title: Read Replicas
order: 062
description: Improve your Gigalixir Standard Tier database performance with read replicas.
permalink: /docs/database/read-replicas
aliases: 
  - /docs/database/read-replicas.html
---

Read replicas databases offer a range of advantages that bolster database performance, availability, and scalability.
You can have an unlimited amount of read replicas per Standard tier database.



## List Read Replicas for a Database

First you will need to get the ID for your database:
``` bash
gigalixir pg
```

Now you can list the read replicas:
``` bash
gigalixir pg:read_replicas -d $DATABASE_ID
```



## Creating Read Replicas

Add a new read replica with:
``` bash
gigalixir pg:read_replicas:create -d $DATABASE_ID
```

By default the read replica will have the same size as the primary.  
You can specify a size on creation if you wish:
``` bash
gigalixir pg:read_replicas:create -d $DATABASE_ID --size=$SIZE
```



## Enabling High Availability

Read replicas are created as single instances.
High availability can be enabled with the pg:scale command:
``` bash
gigalixir pg:scale -d $DATABASE_ID --high_availability=enabled
```

{% include database-scaling-downtime-warning.md %}

Equally, high availability can be disabled with:
``` bash
gigalixir pg:scale -d $DATABASE_ID --high_availability=disabled
```

See [High Availability Pricing](#ha-pricing) for cost.



## Scaling

After you create your read replica, you can scale it with the same command as for the primary with:
``` bash
gigalixir pg:scale -d $READ_REPLICA_ID --size=$SIZE
```
{% include database-scaling-downtime-warning.md %}
{% include database-sizes.md %}



## Working with Read Replicas in Elixir
{: #integration }

When it comes time to utilize your read replicas, Ecto makes it easy.
We suggest you create a config variable for each Read Replica:
``` bash
gigalixir config:set READ_REPLICA1_URL=ecto://user:pass@host:5432/database
```

You can get your read replica URL from the listing:
``` bash
gigalixir pg:read_replicas -d $PRIMARY_DATABASE_ID
```
Just be sure to switch from `postgresql://` to `ecto://`.


Next we update our Repo module to understand there is a read replica:
``` elixir
# lib/fancy_app/repo.ex

defmodule FancyApp.Repo do
  use Ecto.Repo,
    otp_app: :fancy_app,
    adapter: Ecto.Adapters.Postgres

  def replica, do: FancyApp.Repo.Replica
end

defmodule FancyApp.Repo.Replica do
  use Ecto.Repo,
    otp_app: :fancy_app,
    adapter: Ecto.Adapters.Postgres,
    default_dynamic_repo: FancyApp.Repo.Replica,
    read_only: true
end
```

Now we need to add the configuration:
``` elixir
# config/prod.exs (or runtime.exs, or releases.exs)

config :fancy_app, FancyApp.Repo.Replica,
  ssl: true,
  url: {:system, "READ_REPLICA1_URL"},
  pool_size: {:system, "POOL_SIZE"},
  pool_timeout: 10_000
```
If your read replica has a different size than your primary database,
consider using a separate config variable for the `pool_size`.

::: important
::: title
Note
:::
You may have to make considerations in your dev and test environments.
Some users find success in setting the read replica URL to the same URL as the primary in dev and test environments.
:::


Now update the application startup:
``` elixir
# lib/fancy_app/application.ex

def start(_type, _args) do
  ...

  children = [
    FancyApp.Repo,
    FancyApp.Repo.Replica,
    ...
```

With that all done, it is time to utilize the read replica.

``` elixir
  def list_usernames do
    from(u in User, select: u.username)
    |> Repo.replica().all()
  end
```

See [Ecto's Read Replica Documenation](https://hexdocs.pm/ecto/replicas-and-dynamic-repositories.html)
for further details.



## Strategies for Utilizing Read Replicas
{: #strategies }

Some users will setup their read replicas to perform **all** of their read operations,
reserving the primary database for write only operations.
In this scenario it can be useful to make a ReplicaPool module that will round robin
requests to the various read replicas.

If you are running multiple application replicas and multiple read replicas,
some users will assign a read replica to each application replica.

Another strategy for utilizing a read replica is to use the replica purely for intensive queries or
long running jobs.

Read replicas can also be utilized to isolate external applications or providers to read-only operations
that will not affect your primary database.

Read replicas can take a little planning, but the payoff can be huge.


## Users and Roles
{: #permissions }

Read replica are exact copies of the primary database.
They will contain the same users, permissions, views, etc as the primary database.
Any users, roles, views, etc that you may want in the read replica will need to be setup in the primary database.



## Restarting a Read Replica
{: #restarting }

If you need to restart your read replica,
[contact us](/docs/troubleshooting#supporthelp) and we'll help you out.



## Destroying Read Replicas
{: #destroying }

When you no longer need the read replica, you can destroy it with:
``` bash
gigalixir pg:destroy -d $READ_REPLICA_ID
```



## Read Replica Pricing
{: #pricing }

{:.narrow-table}
| Size | Price / Month | RAM    | Connection Limit |
| ---: | ------------: | -----: | :--------------: |
| 0.6  | \$25          | 0.6 GB | 25               |
| 1.7  | \$50          | 1.7 GB | 50               |
| 4    | \$100         | 4 GB   | 100              |
| 8    | \$200         | 8 GB   | 200              |
| 16   | \$400         | 16 GB  | 250              |
| 32   | \$800         | 32 GB  | 300              |
| 48   | \$1200        | 48 GB  | 350              |
| 64   | \$1600        | 64 GB  | 400              |
| 96   | \$2400        | 96 GB  | 500              |



## High Availability Pricing
{: #ha-pricing }

Adding high availabilty doubles the cost of the read replica.
Prices are prorated to the second.
See [Read Replica Pricing](#pricing) for the undoubled prices per size.



## Scaling Times

Scaling a standard tier database requires a restart of the database.
Below is a list of the typical operation times for database sizes.

{:.narrower-table}
| Size | Typical Scaling Time |
| :--: | :------------------: |
| 0.6  | 10 - 25 minutes      |
| 1.7  | 10 - 25 minutes      |
| 4+   | 4 - 8 minutes        |
