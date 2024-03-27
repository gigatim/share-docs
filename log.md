---
layout: doc
toc: true
title: Logging
description: Gigalixir logging help for tailing, forwarding, and other logging information for the Giglaixir platform.
order: 080
permalink: /docs/log
aliases: 
  - /docs/log.html
---

## How to Tail Logs

You can tail logs in real-time - aggregated across all containers - using the following command.

``` bash
gigalixir logs
```

## How to Forward Logs Externally

If you want to forward your logs to another service such as [Logflare](https://logflare.app/){:target="\_blank"}, [Timber](https://timber.io){:target="\_blank"} or [PaperTrail](https://papertrailapp.com/){:target="\_blank"}, you'll need to set up a log drain. 

We support HTTPS and syslog drains. 

To create a log drain, run:

``` bash
gigalixir drains:add $URL
# e.g. gigalixir drains:add https://api.logflare.app/logs/logplex?api_key=$API_KEY&source=$SOURCE_ID
# e.g. gigalixir drains:add https://user:$TIMBER_API_KEY@logs.timber.io/sources/$TIMBER_SOURCE_ID/frames
# e.g. gigalixir drains:add syslog+tls://logs123.papertrailapp.com:12345
```

To show all your drains, run:

``` bash
gigalixir drains
```

To delete a drain, run:

``` bash
gigalixir drains:remove $DRAIN_ID
```
