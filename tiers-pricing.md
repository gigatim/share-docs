---
layout: doc
toc: true
title: Tiers & Pricing
description: Explanation on the different Gigalixir tiers, detailed tier
  comparison, pricing details, database sizes and pricing, replica sizing and
  usage limits | Gigalixir
order: 120
permalink: /docs/tiers-pricing
aliases:
  - /docs/tiers-pricing.html
---
## Tiers

::: important
::: title
Important
:::
Tiers are applicable at the account level, not at the app or database level. 

Therefore, a Free Tier account can not have a payable set of apps or databases, nor can a Standard Tier account have a free set of apps.
:::

We try and keep things very simple at Gigalixir, so we only offer two account / pricing tiers.

The **Free Tier** and the **Standard Tier**.

## Free Account Tier

The Free Tier is, well...free! It also requires no credit card and the usage is free.

 *But,* your apps are limited to one instance up to size 0.5 and one free tier database. 

 Although the database is free, it is really not suitable for production use. You are limited to 2 connections, 10,000 rows, and no backups. Idle connections are terminated after 5 minutes.

Also, on the Free Tier, if you haven't deployed an app for over 30 days, we
will send you an email to remind you to keep your account active. If you do not keep your account active, your app may be scaled down to 0 replicas. 

We know this isn't ideal, but we think it is better than sleeping instances in most cases. 

We really want to be here as an asset to the global developer community with our Free Tier. We designed this tier to be of service to students, educators, testers, and new engineers, so we appreciate your understanding of the above situation as the free tier does cost us a lot of money to run for the developer community.

## Standard Account Tier

The Standard tier is ideal for development, testing, and running production applications.

CPU, bandwidth, and power are still free on the Standard Account Tier, however you pay $5 for every 100MB of memory per month ($0.05/MB/Month/Replica).

Please check [Database Pricing](/docs/database/standard-tier#pricing) for standard tier database costs.

## Tier Comparison

| Instance Feature      | FREE Tier  | STANDARD Tier |
| --------------------- | :--------: | :-----------: |
| Zero-downtime deploys |    YES     |      YES      |
| Websockets            |    YES     |      YES      |
| Automatic TLS         |    YES     |      YES      |
| Log Aggregation       |    YES     |      YES      |
| Log Tailing           |    YES     |      YES      |
| Hot Upgrades          |    YES     |      YES      |
| Remote Observer       |    YES     |      YES      |
| No Connection Limits  |    YES     |      YES      |
| No Daily Restarts     |    YES     |      YES      |
| Custom Domains        |    YES     |      YES      |
| Postgres-as-a-Service |    YES     |      YES      |
| SSH Access            |            |      YES      |
| Vertical Scaling      |            |      YES      |
| Horizontal Scaling    |            |      YES      |
| Clustering Multiple   |            |      YES      |
| Apps Team Permissions |            |      YES      |
| No Inactivity Checks  |            |      YES      |
| Dedicated Ingress     |            |   AVAILABLE   |

For a comparison of database tiers, see [Database Tier Comparison](/docs/database#tier-comparison).

## Gigalixir vs Heroku Feature Comparison

| Feature                 | Gigalixir FREE Tier | Gigalixir STANDARD Tier | Heroku Free | Heroku Standard | Heroku Performance |
| ----------------------- | :------: | :------: | :------: | :------: | :---: |
| Web sockets             | YES                 | YES                     | YES         | YES             | YES                |
| Log Aggregation         | YES                 | YES                     | YES         | YES             | YES                |
| Log Tailing             | YES                 | YES                     | YES         | YES             | YES                |
| Custom Domains          | YES                 | YES                     | YES         | YES             | YES                |
| Postgres-as-a-Service   | YES                 | YES                     | YES         | YES             | YES                |
| No sleeping             | YES                 | YES                     |             | YES             | YES                |
| Automatic TLS           | YES                 | YES                     |             | YES             | YES                |
| Preboot                 | YES                 | YES                     |             | YES             | YES                |
| Zero-downtime deploys   | YES                 | YES                     |             |                 |                    |
| SSH Access              | YES                 | YES                     |             |                 |                    |
| Hot Upgrades            | YES                 | YES                     |             |                 |                    |
| Remote Observer         | YES                 | YES                     |             |                 |                    |
| No Connection Limits    | YES                 | YES                     |             |                 |                    |
| No Daily Restarts       | YES                 | YES                     |             |                 |                    |
| Flexible Instance Sizes |                     | YES                     |             |                 |                    |
| Clustering              |                     | YES                     |             |                 |                    |
| Horizontal Scaling      |                     | YES                     |             | YES             | YES                |
| Built-in Metrics        |                     |                         |             | YES             | YES                |
| Threshold Alerts        |                     |                         |             | YES             | YES                |
| Dedicated Instances     |                     |                         |             |                 | YES                |
| Autoscaling             |                     |                         |             |                 | YES                |

## Pricing Details

In the free tier, everything is no-credit-card free. Once you upgrade to
the standard tier, you pay $10 for every 200MB of memory per month
($0.05/MB/Month/Replica). 

CPU, bandwidth, and power are free.

See our [cost estimator](/pricing) to calculate how
much you should expect to pay each month. 

Please keep reading for exactly how we compute your bill.

Every month we calculate the number of `replica-size-seconds` used and charge your credit card.

`replica-size-seconds` is how many replicas you ran multiplied by the size of each replica multiplied by how many seconds they were run. This is aggregated across all your apps and is prorated to the second.

For example, if you ran a single replica of size 0.5 (500 MB) for the entire month, your bill will be:

```bash
(1 replica) * (500 MB) * (1 month) * (0.05 $/MB-Month-Replica) = $25.00
```

If you ran a single replica of size 1.0 (1000 MB) for 10 days, then scaled it up to 3 replicas, then 10 days later scaled the size up to 2.0 and it was a 30-day month, then your bill would be

```bash
[   (1 replica)  * (1000 MB) * (1/3 month)
  + (3 replicas) * (1000 MB) * (1/3 month)
  + (3 replicas) * (2000 MB) * (1/3 month) ]
* (0.05 $/MB-Month-Replica) = $166.67
```

For database pricing, see [Database Pricing](/docs/database/standard-tier#pricing).

## Database Sizes & Pricing

For information on database sizes and pricing, see:

* [Standard Tier Pricing](/docs/database/standard-tier#pricing)

  * [High Availability Pricing](/docs/database/standard-tier#ha-pricing)
  * [Read Replica Pricing](/docs/database/read-replicas#pricing)
* [Free Tier Databases](/docs/database/free-tier)

## Dedicated Ingress Pricing

For information on dedicated ingress pricing, see [Dedicated Ingress Pricing](/docs/dedicated-ingress#pricing).

## Replica Sizing

* A replica is a docker container where your app runs.
* Replica sizes are available in increments of 0.1 between 0.2 and
    384, but for the higher sizes you'll need to
    [contact us](/docs/troubleshooting#supporthelp) first.
* 1 size unit is 1GB memory, 1 CPU share, and 564mbps egress
    bandwidth.
* 1 CPU share is 200m as defined using [Kubernetes CPU requests](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-cpu){:target="_blank"}
    or roughly 20% of a core guaranteed.

  * If you are on a machine with other containers that don't use
        much CPU, you can use as much CPU as you like.
* Memory is defined using [Kuberenetes memory requests](https://kubernetes.io/docs/concepts/configuration/manage-compute-resources-container/#meaning-of-memory){:target="_blank"}.

  * If you are on a machine with other machines that don't use much memory, you can use as much memory as you like.
* Memory and CPU sizes can not be adjusted separately.

## Why was my App Scaled Down to 0?

On the **Free Tier**, apps are scaled down to 0 if there have been no deploys for 30 days. We send a warning email after 23 days. To prevent this from happening, make sure you either deploy often or upgrade to the **Standard Tier**.

## Custom Domain Pricing

Gigalixir allows custom domains for each application. Up to 100 custom domains are included free for each application. 

Additional custom domains can be purchased by [contacting us](/docs/troubleshooting#supporthelp).

## Limits

Gigalixir is designed for Elixir/Phoenix apps and it is common for Elixir/Phoenix apps to have many connections open at a time, as well as, have connections open for long periods of time. 

Because of this, we do ***not*** limit the number of concurrent connections or the duration of each connection [^1] [^2].

We also know that Elixir/Phoenix apps are designed to be long-lived and potentially store state in-memory, so we do not restart replicas arbitrarily. 

In fact, replicas should not restart at all, unless there is an extenuating circumstance that requires a restart. 

For apps that require extreme high availability, we suggest that your app be able to handle node restarts just as you would for any app not running on Gigalixir.

That said, we do have a number of limits in order to prevent abuse which are listed below. If you need to request a higher limit, [contact us](/docs/troubleshooting#supporthelp) and we'll do our best to accomodate you.

| Resource      | Limit  |
|  ------------ |  :---: |
| Log Drains    | 5      |
| Apps          | 100    |
| SSH Keys      | 50     |
| Collaborators | 25     |
| Config Vars   | 32kb   |
| Slug Size     | 500mb  |
| Repo Size     | 1gb    |
| Build Time    | 30m    |
| Disk          | 10gb   |
| Bandwidth     | 1tb/mo |

[^1]: Because Gigalixir runs on Google Compute Engine, you may bump into an issue with connections that stay idle for 10m. For more information and how to work around it, see <https://cloud.google.com/compute/docs/troubleshooting>

[^2]: We do have a timeout of 60 minutes for connections after an nginx configuration reload. If you have a long-lived websocket connection and our nginx configuration is reloaded, the connection will be dropped 60 minutes later. Unfortunately, nginx reloads happen frequently under Kubernetes.

## Exceeding Usage Limits

**What happens when an app exceeds the memory limit, even briefly**? 

When you go over the memory limit on a replica, your pod will get [OOMKilled](/elixir/help/memory/what-happens-exceed-memory-limit) and restarted.

Alhtough not instantaneous, the timing is pretty close to it.

**What happens when an app exceeds the memory limit for a sustained period of time**?

Alhtough not instantaneous, When you go over the memory limit on a replica, your pod will get [OOMKilled](/elixir/help/memory/what-happens-exceed-memory-limit) and restarted.

This will continue until the memory limit is increased for your service.

**What happens if an app exceed our CPU limit, even briefly**? 

CPU cycles are throttled, so should not see your application shutdown directly related to CPU usage.  What you will experience is that your app will get slower.

**What happens if an app exceeds the CPU limit for a sustained period of time**?

CPU cycles are throttled, your app will stay in a slower state for the duration of the exceeded limit. 

**hen we go over the limits, does this show up in our log drain so that we can trigger alerts/notifications to our team**?

The errors would show up as an OOMKilled status, which shows up in the logs, allowing you to trigger alerts and notifications.