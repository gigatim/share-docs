---
layout: doc
toc: true
title: Billing
description: How to change your Gigalixir Credit Card, check Gigalixir usage, retrieve down prior invoices, and Gigalixir's money back guarantee. 
order: 130
permalink: /docs/billing
aliases: 
  - /docs/billing.html
---

## How to Change Your Credit Card

To change your credit card, run:

``` bash
gigalixir account:payment_method:set
```

## How to View the Current Period's Usage

To see how many replica-size-seconds you've used so far this month, run:

``` bash
gigalixir account:usage
```

The amount displayed has not been charged yet, since thcharges are run at the end of each billing cycle.

## How to View the Current Active Usage

To see your current usage rate, run

``` bash
gigalixir account:usage:running
```

The amount displayed is the amount your current usage would yield every billing cycle if left untouched.

## How to View Previous Invoices

To see all your previous period's invoices, run:

``` bash
gigalixir account:invoices
```
