---
layout: doc
toc: true
title: The Gigalixir Command-Line Interface
description: How to use the Gigalixir Command-Line Interface (CLI) for encryption, authentication, and error reporting | How to Install and upgrade the CLI | Open Source
order: 200
permalink: /docs/cli
aliases: 
  - /docs/cli.html
---
The Gigalixir Command-Line Interface or CLI is a tool you install on your local machine to control Gigalixir.

## How to Install the CLI

See [install the CLI](/docs/getting-started-guide#install-the-command-line-interface){:target="\_blank"}.

There is also an Arch AUR Package here: [https://aur.archlinux.org/packages/gigalixir-cli](){:target="\_blank"}

If you're interested in creating a macOS Brew formula, [contact us!](/docs/troubleshooting#help){:target="\_blank"}

If you're interested in creating an Ubuntu/Debian package, [contact us!](/docs/troubleshooting#help){:target="\_blank"}

## How to Upgrade the CLI

::: tabs
::: group-tab
macOS
``` bash
pip install -U gigalixir --user
```
:::
::: group-tab
Linux
``` bash
pip3 install -U gigalixir --user
```
:::
::: group-tab
Windows
``` bash
pip3 install -U gigalixir --user
```
:::
:::

## Encryption

All HTTP requests made between your machine and Gigalixir's servers are encrypted.

## Conventions

- No news is good news: If you run a command that produces no
     output, then the command probably succeeded.
- Exit codes: Commands that succeed will return a 0 exit code, and
     non-zero otherwise.
- stderr vs stdout: Stderr is used for errors and for log output.
     Stdout is for the data output of your command.

## Authentication

When you login with your email and password, you receive an API key.

This API key is stored in your `~/.netrc`file. Commands generally use your `~/.netrc` file to authenticate with few exceptions.

## Error Reporting

Bugs in the CLI are reported to Gigalixir's error tracking service. Currently, the only way to disable this is by modifying the source code.

[Pull requests](https://github.com/gigalixir/gigalixir-cli/pulls){:target="\_blank"} are also accepted!

## Open Source

The Gigalixir CLI is open source and we welcome pull requests. See [the gigalixir-cli repository](https://github.com/gigalixir/gigalixir-cli){:target="\_blank"}.
