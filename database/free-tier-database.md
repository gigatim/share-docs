---
layout: doc
toc: true
title: Free Tier Database
toc_title: Free Tier
order: 069
description: Get your application off the ground with our free tier databases.
permalink: /docs/database/free-tier
aliases: 
  - /docs/database/free-tier.html
---

Gigalixir offers free postgres databases for users.
The free tier databases are not suitable for production environments.
Please see [Limitations](#limitations) for further details.



## Creating a Free Database
{: #creating }

To create a free tier database, run:
``` bash
gigalixir pg:create --free
```

This command will create the `DATABASE_URL` environment variable if it does not already exist.
Additionally if the `POOL_SIZE` environment variable is not set, this will set it to 2.

::: important
::: title
IMPORTANT
:::
Make sure you set applications `pool_size` in `prod.exs` to utilize the `POOL_SIZE` environment variable.
The free tier database only allows 2 concurrent connections.
:::



## Listing your Databases
{: #listing }

List databases by running:
``` bash
gigalixir pg
```



## Limitations

In free tier, the database is a free multi-tenant postgres database cluster with
shared CPU, memory, and disk, which is not suitable for production use.  

The free tier database is limited to 2 connections, 10,000 rows, and no backups.
Idle connections are terminated after 5 minutes.

Free tier databases attached to applications that are inactive for an extended period of time will be destroyed.

You can only have one free tier database per application.

For a comparison between the free tier and the standard tier databases, see [Database Tiers](/docs/database#tiers).



## Migrating to Standard Tier
{: #upgrading }

If you started out with a free tier database and then upgraded to the standard tier, **we highly recommend you migrate to a standard tier database**. 

The standard tier databases support encryption, backups, extensions, functions, triggers, 
and dedicated cpu, memory, & disk. 
There are no row limits, connection limits, and they are automatically scalable.  
Standard tier databases are encrypted at rest.

For information on upgrading your account, see
[upgrade account](/docs/account#how-to-upgrade-an-account).

Unfortunately, we can't automatically migrate your free tier db to a standard tier db.
You'll have to
{% comment %}
TODO: update these instructions, there is bound to be a better way and we can probably actually help users upgrade
Almost certainly you don't need to delete the free tier database before you create the standard database.
{% endcomment %}
1. `pg_dump` the free database.
2. Delete the free database with `gigalixir pg:destroy --help`. 
  - Note: postgres may make you scale down to 0 app replicas to do this step, so you'll have some downtime.
3. Create the standard tier database with `gigalixir pg:create`.
4. Restore the data with `psql` or `pg_restore`. You can find the url to use with `gigalixir pg` once the standard tier database is created.

Please don't hesitate to [Contact Us](/docs/troubleshooting#supporthelp) if you need help with this migration process.

Standard tier databases still have connection limits but they are much more generous.



## Install a Postgres Extension
{: #postgres-extensions }

::: note
::: title
Note
:::
Free Databases do not support extensions - except for citext - which is
preinstalled. For more information, see [Database Tier Comparison](/docs/database#tier-comparison).
:::



## Destroying a Free Database
{: #destroying }

Delete your free tier database by running:
``` bash
gigalixir pg:destroy -d $DATABASE_ID
```
