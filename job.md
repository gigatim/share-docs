---
layout: doc
toc: true
title: Jobs
description: How to Run one-off Gigalixir Jobs and Tasks in the container your app is running or spin up a new container that runs the command and then destroys itself.
order: 160
permalink: /docs/jobs
aliases: 
  - /docs/jobs.html
  - /docs/job.html
  - /docs/job
---
## How to Run Jobs

There are many ways to run one-off jobs and tasks. You can run them in the container your app is running or you can spin up a new container that runs the command and then destroys itself.

To run a command in your app container, run:

``` bash
gigalixir ps:run $COMMAND
# if you're using distillery, you'll probably want $COMMAND to be something like `bin/app eval 'IO.inspect Node.self'`
# if you're using mix, you'll probably want $COMMAND to be something like `mix ecto.migrate`
```

To run a command in a separate container, run:

``` bash
gigalixir run $COMMAND
# if you're using distillery, you'll probably want $COMMAND to be something like `bin/app eval 'IO.inspect Node.self'`
# if you're using mix, you'll probably want $COMMAND to be something like `mix ecto.migrate`
```

The task is not run on the same node as your app running node. Jobs are killed after five minutes.

If you're using the distillery, note that because we start a separate container to run the job, if you need any applications started such as your `Repo`, use `Application.ensure_all_started/2`.

Also, be sure to stop all applications when done, otherwise your job will never complete and just hang until it times out.

Distillery commands currently do not support passing arguments into the job.

We prepend `Elixir.` to your module name to let the BEAM virtual machine know that you want to run an Elixir module rather than an Erlang module. The BEAM doesn't know the difference between Elixir code and Erlang code once it is compiled down, but compiled Elixir code is namespaced under the Elixir module.

The size of the container that runs your job will be the same size as the app containers and billed the same way, based on replica-size-seconds. (_For more on billing, see [Pricing Details](/docs/tiers-pricing#pricing-details)_).
