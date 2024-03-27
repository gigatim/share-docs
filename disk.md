---
layout: doc
toc: true
title: Disk
description: Understanding the nature of the disk storage, what actions can be performed, the frequency of Gigalixir app regular/occassional restarts and when data is lost
order: 180
permalink: /docs/disk
aliases: 
  - /docs/disk.html
---

## Is the Disk Persistent or Ephemeral?

The short answer is, **Ephemeral**. 

You have full access to read and write from the disk for temporary files, but each time the app restarts, the disk is wiped clean and replaced with a new one. 

This may be sufficient for many use cases, but depends on the requirements of your app. 

Gigalixir app restarts do not happen every 24 hours like they do on Heroku and hot upgrades allow you to deploy with restarting the replica, but there will still be infrequent restarts due to kubernetes upgrades and app crashes.

If you're trying to use something like Mnesia or DETS, feel free to use the disk, but make sure you're okay with losing the data on restart.
