---
layout: doc
toc: true
title: Packages & Dependencies
description: Gigalixir packages and dependencies, what packages are available, installing binaries, and git private repo dependencies.
order: 170
permalink: /docs/dependency
aliases: 
  - /docs/dependency.html
---
## What Packages are Available for My Gigalixir App?

Gigalixir's stacks are based on Heroku's stacks, so anything you find
here, you can find on Gigalixir. [https://devcenter.heroku.com/articles/stack-packages#installed-ubuntu-packages](https://devcenter.heroku.com/articles/stack-packages#installed-ubuntu-packages){:target="\_blank"}

To find what stack you are on, run `gigalixir apps:info` or `gigalixir ps`. If you are on gigalixir-18, check the heroku-18 column. This is the same for all other stacks.

You can also explore and verify by SSH'ing into your container. For example:

``` bash
gigalixir ps:ssh
convert --help
```

## How do I Install Extra Binaries I Need for my App?

The process is different if you are using releases (_Distillery, Elixir releases_) or Mix. 

We recommend switching to Mix mode as it's much easier. 

To switch to mix mode, see [How Do I Switch to Mix Mode?](/docs/modify-app#how-do-i-switch-to-mix-mode).

In mix mode, all you have to do is add the relevant, buildpack to your `.buildpacks` file that is likely located at the top. 

Also, make sure you also have the required Elixir, Phoenix, and mix buildpacks. For example, if you need Rust installed, your `.buildpacks` file might look like:

``` bash
https://github.com/emk/heroku-buildpack-rust
https://github.com/HashNuke/heroku-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-mix.git
```

For Rust specifically, also be sure to run `echo "RUST_SKIP_BUILD=1" > RustConfig`
since you just need the Rust binaries, and don't want to build a Rust project.

In Mix mode, the entire build folder is packed up and shipped to your run container which means it will pack up the extra binaries you've installed and any .profile.d scripts needed to make them available.

Voila, that's it!

If you want to continue using Distillery, you need to manually figure out which folders and files need to be packed into your release tarball and copy them over using Distillery overlays. See [https://github.com/bitwalker/distillery/blob/master/docs/extensibility/overlays.md](https://github.com/bitwalker/distillery/blob/master/docs/extensibility/overlays.md){:target="\_blank"}

If you are using Elixir releases, you also need to manually figure out which folders and files you need to be packed into your release tarball and copy them over using an extra \"step\". See [https://hexdocs.pm/mix/Mix.Tasks.Release.html#module-steps](https://hexdocs.pm/mix/Mix.Tasks.Release.html#module-steps){:target="\_blank"}

## How Do I Use a Private Git Dependency?

If you want to use a private git repository as a dependency in `mix.exs`, our recommended approach is to use the netrc buildpack found at [https://github.com/timshadel/heroku-buildpack-github-netrc](https://github.com/timshadel/heroku-buildpack-github-netrc){:target="\_blank"}

To use the buildpack, insert it in your `.buildpacks` file above the Elixir and Phoenix buildpacks. For example, if you are using distillery, your `.buildpacks` file will look like:

``` bash
https://github.com/timshadel/heroku-buildpack-github-netrc.git
https://github.com/HashNuke/heroku-buildpack-elixir
https://github.com/gigalixir/gigalixir-buildpack-phoenix-static
https://github.com/gigalixir/gigalixir-buildpack-distillery.git
```

Next, create a personal access token by following
[https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/](https://help.github.com/articles/creating-a-personal-access-token-for-the-command-line/){:target="\_blank"}

Just make sure you give the token \"repo\" access so that it can access your private repository.

Add your personal access token as a config var by running:

``` bash
gigalixir config:set -a $APP_NAME GITHUB_AUTH_TOKEN="$GITHUB_TOKEN"
```

The last step is to add the dependency to your `mix.exs` file. Add it as you would any other git dependency, but be sure you use the HTTPS url and not the SSH url. For example:

``` elixir
{:foo, git: "https://github.com/jesseshieh/foo.git", override: true}
```

That should be it.

Alternatively, you could also put your Github username and personal access token directly into the git url, but it's generally not a good idea to check in secrets to source control. 

You could use `System.get_env` interpolated inside the git url, but then you run the risk of the secrets getting saved to `mix.lock`.
