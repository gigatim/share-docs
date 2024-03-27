---
layout: doc
toc: true
title: Database Backup and Recovery
toc_title: Backup and Recovery
order: 063
description: Information about Gigalixir Standard Tier database automatic backups.
permalink: /docs/database/backup-and-recovery
aliases: 
  - /docs/database/backup-and-recovery.html
---


Standard tier databases include 7 days of automatic daily backups.
The information below will help in the event you need to restore to a prior point.



## Restoring a Database Backup

We take automatic backups every day and keeps seven backups available.

First, get your database ID by running:
``` bash
gigalixir pg
```

View what backups you have available by running:
``` bash
gigalixir pg:backups -d $DATABASE_ID
```

Find the backup ID you want.

Recover the backup with:
``` bash
gigalixir pg:backups:restore -d $DATABASE_ID -b $BACKUP_ID
```

This can take a while. Sometimes over ten minutes. To check the status, run:
``` bash
gigalixir pg
```


## Restoring to a Separate Database Instance

If you want to keep your current data, but would like your backup restored to a new database,
[contact us for help](/docs/troubleshooting#supporthelp).
