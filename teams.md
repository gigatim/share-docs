---
layout: doc
toc: true
title: Teams & Organizations
description: Gigalixir Team allows you to share access with collaborators and change the ownership of apps via the Giglaixir Support channel.
order: 100
permalink: /docs/teams
aliases: 
  - /docs/teams.html
---

## How Do I Give Someone Access to my App?

If you work in a team, you'll probably want to collaborate with other users. 

With Gigalixir's access permissions, you can grant access using the commands below. They'll be able to deploy & rollback, manage configs, SSH, remote_console, observer, hot upgrade, and scale.

First - and often skipped - they need to [sign up for their own Gigalixir account](https://console.gigalixir.com/#/register){:target="\_blank"}. 

Then run the command below to give them access.

``` bash
gigalixir access:add $USER_EMAIL
```

To see, who has access, run:

``` bash
gigalixir access
```

To deny access to a user, run:

``` bash
gigalixir access:remove $USER_EMAIL
```

If you don't have access to the CLI and want to modify access, [contact us](mailto:support@gigalixir.com){:target="\_blank"} and we'll help you get your user setups all straightened out.

## Who is Financially Responsible with Multiple Users?

The \"owner\", the user that created the app, is responsible for the Gigalixir usage bill each month.

For organizations, we recommend creating a conceptual \"organization account\" with Giglaixir that is upgraded to the standard tier with the billing information on file. Then, create individual accounts for all developers and grant access to all contributors.

That helps keep the billing separate from the individual team members and contributors.

## How Do I Change the Owner of an App?

We can do that, no problem, it's just not something we allow in the CLI or GUI for security reasons...Therefore, [Contact us](/docs/troubleshooting#supporthelp) and we'll
help you change ownership and verify billing so service is uninterrupted.

Make sure prior to contacting us that the new owner has a Gigalixir account and is on the same payment tier as the current account to help expedite the request.
