---
layout: doc
toc: true
title: SSH, Remote Console & Observer
description: Configuring, managing & running SSH keys & other options, how to drop into a Remote Console, run Distillery commands & launch the Observer tools for Gigalixir.
order: 090
permalink: /docs/runtime
aliases: 
  - /docs/runtime.html
---
## Managing SSH Keys

In order to SSH, run Remote Observer, Remote Console, etc, you need to set up your SSH keys. 

It could take up to a minute for the SSH keys to update in your containers.

``` bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_ed25519.pub)"
```

If you don't have an `id_ed25519.pub` file, follow [this
guide](https://help.github.com/articles/generating-a-new-ssh-key-and-adding-it-to-the-ssh-agent/){:target="\_blank"} to create one. 

Or if you have an RSA key already, run `gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"` instead.

To view your SSH keys

``` bash
gigalixir account:ssh_keys
```

To delete an SSH key, find the key's id and then run delete the key by
id.

``` bash
gigalixir account:ssh_keys:remove $ID
```

## How to SSH into a Production Container

If your app is running, but not behaving, SSH'ing to the app might give you some insight into what is happening. 

A major caveat, though, is that **the app has to be running**. 

If the app isn't running, then it isn't passing health checks, and we'll keep restarting the entire container. Obviously, you won't be able to SSH into a container that is restarting non-stop. 

If your app isn't running, try taking a look at [Troubleshooting](/docs/troubleshooting){:target="\_blank"}.

To SSH into a running production container:

First, add your public SSH keys to your account. 

_(For more information on managing SSH keys, see [Managing-SSH-Keys](/docs/runtime#managing-ssh-keys){:target="\_blank"})_.

``` bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"
```

Next, use the following command to SSH into a live production container.

If you are running multiple containers, this will put you in a random
container. 

We do not yet support specifying which container you want to SSH into and, in order for this work, you must add your public SSH keys to your account.

``` bash
gigalixir ps:ssh
```

## How to specify SSH key or other SSH options

The `-o` option lets you pass in arbitrary options to `ssh`. You can use this option to specify which SSH key to use.

``` bash
gigalixir ps:ssh -o "-i ~/.ssh/id_rsa"
```

If you have multiple SSH keys on your machine, you may need to explicitly specify which one the Gigalixir CLI should use when connecting. 

If you get a _Permission denied (publickey)_ error when attempting to run commands through the CLI but your `git push gigalixir main` (_or equivalent_) succeeds, first try specifying the SSH key you want to use with the option above.

To avoid having to specify the key file on each run, set the `GIGALIXIR_IDENTITY_FILE` to the path to your private key.

``` bash
export GIGALIXIR_IDENTITY_FILE=$HOME/.ssh/gigalixir
```

You can use `-o` to specify any option or
options to `ssh`.

## How to Drop into a Remote Console

To get a console on a running production container, first, add your public SSH keys to your account. For more information on managing SSH keys, see [Managing-SSH-Keys](/docs/runtime#managing-ssh-keys){:target="\_blank"}.

``` bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"
```

Then run this command to drop into a Remote Console:

``` bash
gigalixir ps:remote_console
```

## How to Run Distillery Commands

Our CLI provides all the commands Distillery provides such as ```ping```, ```rpc```, ```command```, and ```eval```. [Launching a Remote Console](#how-to-drop-into-a-remote-console){:target="\_blank"} is just a special case of this. 

To run a Distillery command, run the command below. 

For a complete list of commands, see [Distillery's boot.eex](https://github.com/bitwalker/distillery/blob/master/priv/templates/){:target="\_blank"}.

``` bash
gigalixir ps:distillery $COMMAND
```

## How to Launch a Remote Observer

To connect a Remote Observer, you need to be using Distillery or Elixir
releases. See [Mix vs Distillery](/docs/modify-app/#mix-vs-distillery-vs-elixir-releases){:target="\_blank"}.

In order to run a Remote Observer, you need to set up your SSH keys. It
could take up to a minute for the SSH keys to update in your containers.

``` bash
gigalixir account:ssh_keys:add "$(cat ~/.ssh/id_rsa.pub)"
```

Because Observer runs on your local machine and connects to a production
node by joining the production cluster, you first have to have
clustering set up. You don't have to have multiple nodes, but you need
to follow the instructions in [cluster your nodes](/docs/cluster#clustering-nodes){:target="\_blank"}.

You also need to have `runtime_tools`
in your application list in your `mix.exs` file. Phoenix 1.3 and later adds it by default, but you
have to add it yourself in Phoenix 1.2.

Your local machine also needs to have `lsof`.

You should also make sure your app has enough memory. Even though observer itself is running on your local machine, the remote machine still needs quite a bit of memory. For a basic app, make sure you have at least 500mb memory (_size 0.5_).

Then, to launch observer and connect it to a production node, run"

``` bash
gigalixir ps:observer
```

The instructions will prompt you for your local sudo password so it can modify iptables rules. This connects to a random container using consistent hashing. We don't currently allow you to specify which container you want to connect to, but it will connect to the same container each time based on a hash of your ip address.

## Monitoring

Gigalixir doesn't provide any monitoring out of the box, you can always use a [Remote Observer](/docs/runtime#how-to-launch-a-remote-observer){:target="\_blank"} to inspect your node.
