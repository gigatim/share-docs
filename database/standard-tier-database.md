---
layout: doc
toc: true
title: Standard Tier Database
toc_title: Standard Tier
order: 061
description: Deploy a production quality database with the Gigalixir Standard tier database.
permalink: /docs/database/standard-tier
aliases: 
  - /docs/database/standard-tier.html
---


Gigalixir's Standard Tier databases are production level databases.
Standard tier databases can grow with your application.
Additionally we offer read replicas and high availability.



## Creating a Standard Tier Database
{: #creating }

To create a standard tier database, run:
``` bash
gigalixir pg:create --size=$SIZE
```

{% include database-sizes.md %}

Give it a moment, it takes a few minutes ⏲️ to provision.
See [Scaling Times](#scaling-times) for details.

You can check the status by running:
``` bash
gigalixir pg
```

The create command will create the `DATABASE_URL` environment variable if it does not already exist.
Additionally if the `POOL_SIZE` environment variable is not set, this will set it to 9.
For more information on properly setting the pool size, see [Pool Size Determination](#pool-size-determination).

You can only have one standard tier database per application.

Backups are automatically configured and happen daily.
We keep 7 days worth of backups, which can be restored.
See [Backup and Recovery](/docs/database/backup-and-recovery) for more details.

Standard tier databases are created as single instances.
High availability can be enabled, see [High Availability](#high-availability).



## Listing your Databases
{: #listing }

List databases by running:
``` bash
gigalixir pg
```



## Scaling Your Database
{: #scaling }

To change the size of your database, run:
``` bash
gigalixir pg:scale -d $DATABASE_ID --size=$SIZE
```

{% include database-sizes.md %}

You can find your database ID by running:
``` bash
gigalixir pg
```

{% include database-scaling-downtime-warning.md %}



## Pool Size Determination

You may also want to adjust your pool_size. We recommend setting the
pool size to (M-6)/(n+1) where M is the max connections and n is the num
app replicas. 

We subtract 6 because Cloud SQL will sometimes - but rarely - use 6 for maintenance purposes. 

We use n+1 because rolling deploys will temporarily have an extra replica during the transition.

For example, if you are running a size 0.6 database with 1 app replica, the pool size should be (25-6)/(1+1)=9.



## High Availability

Databases are created as single instances.
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



## Read Replicas

Read replicas are a great way to improve the performance of your database.
Standard databases can have an unlimited number of read replicas.

See [Read Replicas](/docs/database/read-replicas) for details.



## Backup and Recovery

All standard tier databases are automatically backed up daily.
We store 7 days worth of backups you can restore to.

See [Backup and Recovery](/docs/database/backup-and-recovery) for details.



## Users and Roles
{: #permissions }

On the standard tier, your credentials give you administrative privileges to your database.
You can create users and roles, grant permissions, create views, etc.



## Install a Postgres Extension
{: #postgres-extensions }

First, make sure Google Cloud SQL supports your extension by checking
[their list of extensions](https://cloud.google.com/sql/docs/postgres/extensions){:target="\_blank"}. 

Get a psql console into your database:
``` bash
gigalixir pg:psql $DATABASE_URL
```

Then, install your extension:
``` bash
CREATE EXTENSION foo;
```



## Restarting a Standard Database
{: #restarting }

If you need to restart your database,
[contact us](/docs/troubleshooting#supporthelp) and we'll help you out.



## Destroying a Standard Database
{: #destroying }

::: important
::: title
Warning
:::
Deleting a database also deletes all of its backups. 
Please make sure you backup your data first.
This operation is unrecoverable.
:::

Delete your database by running:
``` bash
gigalixir pg:destroy -d $DATABASE_ID
```



## Database Pricing
{: #pricing }

| Size | Price / Month | RAM    | Rollback Days | Connection Limit | Storage Limit |
| ---: | ------------: | -----: |:------------: | :--------------: | ------------: |
| 0.6  | \$25          | 0.6 GB | 7             | 25               | 25 GB         |
| 1.7  | \$50          | 1.7 GB | 7             | 50               | 50 GB         |
| 4    | \$100         | 4 GB   | 7             | 100              | 100 GB        |
| 8    | \$200         | 8 GB   | 7             | 200              | 200 GB        |
| 16   | \$400         | 16 GB  | 7             | 250              | 400 GB        |
| 32   | \$800         | 32 GB  | 7             | 300              | 800 GB        |
| 48   | \$1200        | 48 GB  | 7             | 350              | 1.2 TB        |
| 64   | \$1600        | 64 GB  | 7             | 400              | 1.6 TB        |
| 96   | \$2400        | 96 GB  | 7             | 500              | 2.4 TB        |

Prices are prorated to the second.

Sizes 0.6 and 1.7 share CPU with other databases. 

All other sizes have dedicated CPU, 1 CPU for every 4 GB of memory. 

For example, size 4 has 1 dedicated CPU and size 64 has 16 dedicated CPUs.
All databases start with 10 GB disk and increase automatically, as needed.

[Contact Us](/docs/troubleshooting#supporthelp) if you are interested in additional sizes.



## High Availability Pricing
{: #ha-pricing }

Adding high availabilty doubles the cost of the database.
Prices are prorated to the second.
See [Database Pricing](#pricing) for the undoubled prices per size.



## Scaling Times

Scaling a standard tier database requires a restart of the database.
Below is a list of the typical operation times for database sizes.

{:.narrower-table}
| Size | Typical Scaling Time |
| :--: | :------------------: |
| 0.6  | 10 - 25 minutes      |
| 1.7  | 10 - 25 minutes      |
| 4+   | 4 - 8 minutes        |
