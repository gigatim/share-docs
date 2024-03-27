---
layout: doc
toc: true
title: Modifying an Existing App to Run on Gigalixir
toc_title: Modify an Existing App
description: |
  Modifying an Existing App to Run on Gigalixir | Whether you have an existing app or you just ran mix phx.new, the goal of this guide is to get your app ready
order: 030
permalink: /docs/modify-app/
aliases: 
  - /docs/modify-app.html
---
## Modifying an Existing App to Run on Gigalixir

Whether you have an existing app or you just ran `mix phx.new`, the goal of this guide is to get your app ready for deployment on Gigalixir.

We assume that you are using Phoenix with Elixir and Gigalixir. If you aren't, feel free to [contact us](mailto:support@gigaixir.com){:target="\_blank"} for help with other frameworks.

However, as long as your app is serving HTTP traffic on `$PORT`, you should be fine.

::: important
::: title
Important
:::
If you have an umbrella app, be sure to *also* see
[How do I deploy an umbrella app?](/docs/deploy#how-do-i-deploy-an-umbrella-app).
:::

## Elixir Releases vs Mix vs Distillery

{% include distillery-deprecation.md %}

Probably the hardest part of deploying Elixir is choosing which method of deploying you prefer, but don't worry, it's easy to change your mind later and switch.

We typically recommend Elixir Releases because it is easy to set up and unlocks the most important features, like Observer.

Here is a comparison table to help you choose.

| Feature                   | Elixir Releases |  Mix  | Distillery |
| ------------------------- | :-------------: | :---: | :--------: |
| Hot Upgrades              |                 |       |    YES     |
| Remote Observer           |       YES       |       |    YES     |
| Mix Tasks                 |                 |  YES  |            |
| Built-in to               |       YES       |  YES  |            |
| Elixir                    |                 |       |            |
| Easy Configuration ¹      |                 |  YES  |            |
| Clustering                |       YES       |  YES  |    YES     |
| gigalixir ps:migrate Fast |       YES       |  YES  |    YES     |
| Fast startup ²            |       YES       |       |    YES     |
| Hidden Source Code ³      |       YES       |       |    YES     |

If you choose Elixir releases, see
[Modifying an App for Gigalixir using Elixir Releases](/docs/modify-app/releases)
or
[Phoenix Deploy with Releases](/docs/getting-started-guide/phoenix-releases-deploy).

If you want to use Mix, see
[Modifying an App for Gigalixir using Mix](/docs/modify-app/mix)
or
[Phoenix Deploy with Releases](/docs/getting-started-guide/phoenix-mix-deploy).

If you choose Distillery, see
[Modifying an App for Gigalixir using Distillery](/docs/modify-app/distillery)
or
[Phoenix Deploy with Releases](/docs/getting-started-guide/phoenix-distillery-deploy).

¹ *We say easy configuration here because some customers get confused about the difference between prod.exs and releases.exs. Distillery can be even more confusing with its `REPLACE_OS_VARS` syntax.*

² *Due to smaller slug sizes.*

³ *Mix deploys the source code to the runtime container rather than just the compiled BEAM files*

## How Do I Switch to Modes?

Mix mode is the default, but we automatically detect automatically detect and switch you to Elixir Releases mode if you have a `config/releases.exs` file.
To override that behavior, you just need to delete that file.

If you are newer versions of Elixir, you may have a `config/runtime.exs` file, but not a `config/releases.exs`.
If you wish to use Elixir Releases, just add an empty `config/runtime.exs` file to your repoistory.

We also automatically switch to Distillery mode if you have a `rel/config.exs` file.
To override that behavior, you just need to delete that file.

You can also specify your buildpacks specifically.
You can find the default buildpacks for each mode in these docs:

* [Elixir Releases](/docs/modify-app/releases#specify-buildpacks-optional)
* [Mix](/docs/modify-app/mix#specify-buildpacks-optional)
* [Distillery](/docs/modify-app/distillery#specify-buildpacks-optional)
