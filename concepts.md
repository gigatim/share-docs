---
layout: doc
toc: true
title: Concepts
description: How Does Gigalixir Work | A detailed look into Components,
  Concepts, Deploys, How SSL/TLS Works, Hot Upgrades, Benchmarks and Security |
  Gigalixir Concepts
order: 210
permalink: /docs/concepts
aliases:
  - /docs/concepts.html
---
## How Does Gigalixir Work?

When you deploy an app on Gigalixir, you `git push` the source code to a build server. The build server compiles the code and assets and generates a standalone tarball we call
a **slug**. 

The controller then combines the slug and your app configuration into a **release**. The release is deployed to run containers which actually run your app.

![image](/assets/images/deploy.png)

When you update a config, we encrypt it, store it, and combine it with the existing slug into a new release. The release is deployed to run containers.

![image](/assets/images/config.png)

### Components

* *Build Server*: This is responsible for building your code into a
    release or slug.
* *API Server / Controller*: This is responsible for handling all
    user requests such as scaling apps, setting configs, etc. It is
    also responsible for deploying the release into a run container.
* *Database*: The database is where all of your app configuration is
    stored. Configs are encrypted due to their sensitive nature.
* *Logger*: This is responsible for collecting logs from all your
    containers, aggregating them, and streaming them to you.
* *Router*: This is responsible for receiving web traffic for your
    app, terminating TLS, and routing the traffic to your run
    containers.
* *TLS Manager*: This is responsible for automatically obtaining TLS
    certificates and storing them.
* *Kubernetes*: This is responsible for managing your run
    containers.
* *Slug Storage*: This is where your slugs are stored.
* *Observer*: This is an application that runs on your local machine
    that connects to your production node to show you everything you
    could ever want to know about your live production app.
* *Run Container*: This is the container that your app runs in.
* *Command-Line Interface*: This is the command-line tool that runs
    on your local machine that you use to control Gigalixir.

### Concepts

* *User*: The user is you. When you sign up, we create a user.
* *API Key*: Every user has an API Key which is used to authenticate
    most API requests. You get one when you login and you can
    regenerate it at any time. It never expires.
* *SSH Key*: SSH keys are what we use to authenticate you when
    SSHing to your containers. We use them for Remote Observer, remote
    console, etc.
* *App*: An app is your Elixir application.
* *Release*: A release is a combination of a slug and a config which
    is deployed to a run container.
* *Slug*: Each app is compiled and built into a slug. The slug is
    the actual code that is run in your containers. Each app will have
    many slugs, one for every deploy.
* *Config*: A config is a set of key-value pairs that you use to
    configure your app. They are injected into your run container as
    environment variables.
* *Replicas*: An app can have many replicas. A replica is a single
    instance of your app in a single container in a single pod.
* *Custom Domain*: A custom domain is a fully qualified domain that
    you control which you can set up to point to your app.
* *Payment Method*: Your payment method is the credit card on file
    you use to pay your bill each month.
* *Permission*: A permission grants another user the ability to
    deploy. Even though they can deploy, you remain the owner and are
    responsible for paying the bill.

### Life of a Deploy

When you run `git push gigalixir`, our
git server receives your source code and kicks off a build using a
pre-receive hook. We build your app in an isolated docker container
which ultimately produces a slug which we store for later. The
buildpacks used are defined in your `.buildpacks` file.

By default, the buildpacks we use include

* <https://github.com/HashNuke/heroku-buildpack-elixir.git>{:target="_blank"}

  * To run mix compile
  * If you want, you can [configure this
        buildpack](https://github.com/HashNuke/heroku-buildpack-elixir#configuration){:target="_blank"}.
* <https://github.com/gigalixir/gigalixir-buildpack-phoenix-static.git>{:target="_blank"}

  * To run mix phx.digest
  * This is only included if you have an assets folder present.
* <https://github.com/gigalixir/gigalixir-buildpack-distillery.git>{:target="_blank"}

  * To run mix release or mix distillery.release
  * This is only run if you have a rel/config.exs file present.
* <https://github.com/gigalixir/gigalixir-buildpack-releases>{:target="_blank"}

  * To run mix release if you are running Elixir 1.9 and using the
            built-in releases

    >
  * This is only run if you have a config/releases.exs file
        present.
        ::: note
        ::: title
        Note
        :::
        Elixir 1.11 adds `config/runtime.exs`. If you use that instead, then you'll want to
        specify buildpacks since we can no longer detect if you want
        releases or mix mode. See
        [Specify Buildpacks (optional)](/docs/modify-app/releases#specify-buildpacks-optional).
        :::
* <https://github.com/gigalixir/gigalixir-buildpack-mix.git>{:target="_blank"}

  * To set up your Procfile correctly
  * This is only run if you *don't* have a rel/config.exs file
        present.

We only build the main or master branch and ignore other branches. When building, we cache compiled files and dependencies so you do not have to repeat the work on every deploy. We support git submodules.

Once your slug is built, we upload it to slug storage and we combine it with a config to create a new release for your app. The release is tagged with a `version` number which you
can use later on if you need to rollback to this release.

Then we create or update your Kubernetes configuration to deploy the app. We create a separate Kubernetes namespace for every app, a service account, an ingress for HTTP traffic, an ingress for SSH traffic, a TLS certificate, a service, and finally a deployment which creates pods and containers.

The container that runs your app is a derivative of [heroku/heroku](https://hub.docker.com/r/heroku/heroku){:target="_blank"}. 

The entrypoint is a script that sets up necessary environment variables including those from your [app configuration](/docs/config#app-configuration){:target="_blank"}. It also starts an SSH server, installs your SSH keys, downloads the current slug, and executes it. 

We automatically generate and set up your Erlang cookie, distributed node name, and Phoenix secret key base for you. We 

also set up the Kubernetes permissions and [libcluster](https://www.gigalixir.com/docs/cluster) selector you need
to [cluster your nodes](/docs/cluster#cluster-your-nodes){:target="_blank"}. We poll for your SSH keys every minute in case they have
changed.

At this point, your app is running. 

The Kubernetes ingress controller is routing traffic from your host to the appropriate pods and terminating SSL/TLS for you automatically. For more information about how SSL/TLS
works, see [How SSL/TLS Works](/docs/concepts#how-ssltls-works).

If at any point, the deploy fails, we rollback to the last known good release.

To see how we do zero-downtime deploys, see [How to Set Up Zero-Downtime Deploys](/docs/deploy#how-to-set-up-zero-downtime-deploys).

### How SSL/TLS Works

We use cert-manager for automatic TLS certificate generation with Let's Encrypt. For more information, see [cert-manager's documentation](https://github.com/jetstack/cert-manager){:target="_blank"}. 

When you add a custom domain, we create a Kubernetes ingress for you to route traffic to your app. cert-manager picks this up, obtains certificates for you and installs them. Our ingress controller then handles terminating SSL traffic before sending it to your app.

### Life of a Hot Upgrade

There is an extra flag you can pass to deploy by hot upgrade instead of a restart. 

You have to make sure you bump your app version in your `mix.exs`. Distillery autogenerates your appup file, but you can supply a custom appup file if you need it. 

For more information, look at the [Distillery appup documentation](https://hexdocs.pm/distillery/upgrades-and-downgrades.html#appups){:target="_blank"}.

```bash
git -c http.extraheader="GIGALIXIR-HOT: true" push gigalixir
```

A hot upgrade follows the same steps as a regular deploy, except for a few differences. In order for distillery to build an upgrade, it needs access to your old app so we download it and make it available in the build container.

Once the slug is generated and uploaded, we execute an upgrade script on each run container instead of restarting. The upgrade script downloads the new slug, and calls [Distillery's upgrade command](https://hexdocs.pm/distillery/walkthrough.html#deploying-an-upgrade){:target="_blank"}.

Your app should now be upgraded in place without any downtime, dropped connections, or loss of in-memory state.

## How Secure is Gigalixir?

Gigalixir takes security very, very seriously.

1. Every app exists in its own Kubernetes namespaces and we use
   Kubernetes role-based access controls to ensure no other apps have
   access to your app or its metadata.
2. Your build environment is fully isolated using Docker containers.
3. Your slugs are authenticated using [Signed URLs](https://cloud.google.com/storage/docs/access-control/signed-urls){:target="_blank"}.
4. All API endpoints are authenticated using API keys instead of your
   password. API keys can be invalidated at any time by regenerating a
   new one.
5. [Remote Console](https://www.gigalixir.com/docs/runtime) and Remote Observer use SSH tunnels to secure
   traffic.
6. Erlang does not encrypt distribution traffic between your nodes by
   default, but you can [set it up to use SSL](http://erlang.org/doc/apps/ssl/ssl_distribution.html){:target="_blank"}. For an
   extra layer of security, we route distribution traffic directly to
   each node so no other apps can sniff the traffic.
7. We use [Stripe](https://stripe.com/){:target="_blank"} to manage payment methods so
   Gigalixir never knows your credit card number.
8. Passwords and app configs are encrypted at rest using
   [Cloak](https://github.com/danielberkompas/cloak){:target="_blank"}.
9. Traffic between Gigalixir services and components is TLS encrypted.
