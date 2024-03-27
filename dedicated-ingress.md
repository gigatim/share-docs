---
layout: doc
toc: true
title: Dedicated Ingress
order: 115
description: Utilize Gigalixir dedicated ingress to improve your application isolation, bandwidth, and latency.
permalink: /docs/dedicated-ingress
aliases: 
  - /docs/dedicated-ingress.html
---


Dedicated Ingress provides you with dedicated load balancers and static IP addresses,
isolating inbound traffic to make your applications more reliable and lowering overall application latency.
Dedicated Ingress setups can be shared among multiple applications in the same region or 
you can opt to isolate applications individually.

With static inbound IP addresses, you can readily integrate with third party services,
allow for destination IP filtering, and easily register an apex domain.

Dedicated Ingress is available for all STANDARD Tier applications.
Contact [Support](http://support.gigalixir.com/support/tickets/new) to get started today.



## Migrating to Dedicated Ingress
{: #migrating}

Setting up a Dedicated Ingress solution is relatively easy.
Our support team will guide you through the process and work with you to ensure minimal downtime.

The first step is to reach out to our
[Support](http://support.gigalixir.com/support/tickets/new) team so we can design a migration plan with you.

You will provide us with a list of applications you would like moved to each dedicated setup.
We will review your DNS configuration with you and determine if there are any steps to be taken before we make the change.
We will also discuss the best time to make changes to your applications traffic.

Once the migration plan is in place, we will build out your dedicated ingress system.
When the time comes to switch to the new system, there will be a brief downtime due to DNS propagation delays.
We can usually keep this under 1 minute with a goal of a handful of seconds.
When planning for the migration, you should allow for a 1-5 minute downtime.



## Pricing
{: #pricing}

{:.narrow-table}
| Tier      | Price / Month | Application Limit |
| --------: | ------------: | ----------------: |
| Dedicated | \$250         | 4                 |

Prices are prorated to the second.



## Reverting to Shared Ingress
{: #destroying }

If you decide you no longer need the dedicated ingress setup, please contact our team to migrate back to shared ingress.
The process is very similar to the setup, and will require a small amount of downtime.
