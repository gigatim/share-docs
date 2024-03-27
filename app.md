---
layout: doc
toc: true
title: App Management
description: How to Create, Name, Delete, Rename, Scale, Check the Status, List Releases, View, Restart Gigalixir Apps and Manage Multiple Apps | Gigalixir App Management
order: 050
permalink: /docs/app
aliases: 
  - /docs/app.html
---

## How to Create an App

To create your app, run the ```create``` command. This will also set up a git remote. 

This **_must_** be run from within a git repository folder. 

An app name will be generated for you, but you can also optionally supply an
app name if you wish using `gigalixir create -n $APP_NAME`. 

There is currently no way to change your app name once it is created. 

If you like, you can also choose which cloud provider and region using the
`--cloud` and `--region` options. 

We currently support `GCP` in `v2018-us-central1` or `europe-west1` and `AWS` in `us-east-1` or `us-west-2`. 

The default is v2018-us-central1 on GCP.

``` bash
gigalixir create
```

## How to Choose a Name for your App

Normally, gigalixir generates a unique name for you automatically, but
if you want, you can specify your app name. You'll need to
[install the CLI](/docs/getting-started-guide#install-the-command-line-interface) and run something like
this

``` bash
gigalixir create -n $APP_NAME
```

Once you deploy, you'll be able to access your app from `https://$APP_NAME.gigalixirapp.com`.

## How to Delete an App

To delete an app, run:

``` bash
gigalixir apps:destroy
```

## How to Rename an App

There is no way to rename an app, but you can delete it and then create
a new one. 

Make sure you migrate over your configs if you delete your app to rename it.

## How to Scale an App

You can scale your app by adding more memory and CPU to each container, also called a replica. 

You can also scale by adding more replicas. 

Both are handled by the following command. For more information about sizing and scaling, see [replica sizing](/docs/tiers-pricing#replica-sizing).

``` bash
gigalixir ps:scale --replicas=2 --size=0.6
```
When scaling an app down, please see [What Happens when an App Scales Down - Graceful Shutdown](/docs/faq#what-happens-when-an-app-scales-down---graceful-shutdown){:target="\_blank"}.

We highly recommend running at least two replicas as soon as possible.
This makes your application more resilient and can help share the load.
See [Adding More Replicas](/blog/considerations-for-adding-more-replicas)
for benefits and considerations of adding more replicas.
Users that solve the "how do I run multiple replicas" concerns early have better success at scaling
their applications as they mature.

## How to Check App Status

To see how many replicas are actually running in production (_compared to how many are actually desired_), run:

``` bash
gigalixir ps
```

## How do I Manage Multiple Apps?

All relevant CLI commands can take an optional `--app_name` flag to specify which app you want to run the command on. Or, if you prefer, you can use the `GIGALIXIR_APP` environment variable instead. 

Without the app name, Gigalixir tries to auto-detect the app name based on your git remotes - which works fine when you only have one app.

## How to List Apps

To see what apps you own and information about them, run the ```apps```
command. 

This will only show you your **_desired_** app configuration. 

To see the **_actual status_** of your app, see [App Status](/docs/app#how-to-check-app-status).

``` bash
gigalixir apps
```

## How to List Releases

Each time you deploy or rollback, a new release is generated. 

To see all your previous releases, run:

``` bash
gigalixir releases
```

## How to View App Activity

We keep a record of each time you deploy, change configs, scale, etc. 

To view the activity history, run:

``` bash
gigalixir apps:activity
```

## How to Restart an App

``` bash
gigalixir ps:restart
```

For hot upgrades, See [Hot Upgrades](/docs/deploy#how-to-hot-upgrade-an-app). We
are working on adding custom health checks.

Restarts should be zero-downtime. See [See How to Set Up Zero-Downtime Deploys](/docs/deploy#how-to-set-up-zero-downtime-deploys).



## Maintenance Mode

Maintenance mode keeps your app running, but stops any incoming web traffic.
While in maintenance mode, requests to your app receive a 404 response.
GET requests will also receive a message about your website undergoing maintenance.

Enable maintenance mode with:
```
gigalixir maintenance:on
```

Disable maintenance mode with:
```
gigalixir maintenance:off
```
