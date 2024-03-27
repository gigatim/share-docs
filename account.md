---
layout: doc
toc: true
title: Account Management
description: Gigalixir account management for signing up, creating an account, deleting an account, and reseting your password or email address. 
order: 040
permalink: /docs/account
aliases: 
  - /docs/account.html
---
## How to Sign Up for an Account

Create an account using the following command. 

``` bash
gigalixir signup
```

It will prompt you for your email address and password. You will have to confirm your email
before continuing. 

Gigalixir's free tier does not require a credit card, but you will be limited to 1 instance with 0.2GB of memory and 1 postgresql database limited to 10,000 rows.

If you wish to signup with your Google account, you can use:

``` bash
gigalixir signup:google
```

## How to Upgrade an Account

::: important
::: title
Important
:::
Tiers are applicable at the account level, not at the app or database level. Therefore, a Free Tier account can not have a payable set of apps or databases, nor can a Standard Tier account have a free set of apps.
:::

The standard tier offers much more than the free tier, see [Tiers](/docs/tiers-pricing#tiers).

The easiest way to upgrade is through the web interface at
<https://console.gigalixir.com/>

To upgrade with the CLI:

``` bash
gigalixir account:upgrade
```

::: important
::: title
NOTE
:::
If you are running a version of the CLI before 1.10, you will need to first set a payment method with
``` bash
gigalixir account:payment_method:set
```
:::


## How to Delete an Account

If you just want to make sure you won't be billed anymore, run:

``` bash
gigalixir apps
```

And for every app listed, run:

``` bash
gigalixir apps:destroy
```

This will make sure you've deleted all domains, databases, etc and you won't be charged in the future for any additional usage.

::: important
::: title
Important
:::
Billing stops for all apps and databases when you destroy the app, however you will still be billed for usage up to the destruction of the app for that current month. 
:::

If you really want to completely destroy your account, run:

``` bash
gigalixir account:destroy
```

## How to Update your Payment Method

With the web interface, visit : <https://console.gigalixir.com/#/account/payment-method> and click on the edit button.

With the CLI, run

``` bash
gigalixir account:payment_method:set
```

## How to Change your Email Address

``` bash
gigalixir account:email:set
```

You will be sent a confirmation email with a link to confirm the email
change. The current email address will be sent an email with a link to
revoke the change.

## How to Change or Reset Your Password

With the web interface, visit
<https://console.gigalixir.com/#/password/reset>

With the CLI, run

``` bash
gigalixir account:password:change
```

If you forgot your password, send a reset token to your email address by
running the following command and following the instructions in the
email.

``` bash
gigalixir account:password:reset
```

## How to Resend the Confirmation Email

With the web interface, visit
<https://console.gigalixir.com/#/confirmation/resend>

With the CLI, run

``` bash
gigalixir account:confirmation:resend
```

## How to Reset your API Key

If you lost your API key or it has been stolen, you can reset it by
running

``` bash
gigalixir account:api_key:reset
```

Your old API key will no longer work and you may have to login again.

## How to Log Out

``` bash
gigalixir logout
```

## How to Log In

``` bash
gigalixir login
```

This modifies your \~/.netrc file so that future API requests will be
authenticated. API keys never expire, but can be revoked.

If you wish to login with your Google account, you can use:

``` bash
gigalixir login:google
```

## How to use Multi-Factor Authentication

MFA, also known as 2-factor authentication or 2FA, this gives your account an
extra layer of security so someone with just your password still won't
be able to login to your account.

To activate MFA with the CLI, first make sure you have version 1.2 or
higher. 

To upgrade your CLI. See [How to Upgrade the CLI](/docs/cli#cli-upgrade). 

Then run:

``` bash
gigalixir account:mfa:activate
```

This logs you out, so re-login:

``` bash
gigalixir login
```

Also, try it out on the web console:
<https://console.gigalixir.com/#/login>

To deactivate, run:

``` bash
gigalixir account:mfa:deactivate
```

To regenerate recovery codes, run

``` bash
gigalixir account:mfa:recovery_codes:regenerate
```

## How to Check Account Status

To check and verify account items - like what account you're logged in as currently, what tier you are on, or how many credits you have available - simply run:

``` bash
gigalixir account
```
