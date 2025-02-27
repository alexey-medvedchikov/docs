---
title: Take Backups with Revision History and Restore from a Point-in-time
summary: Learn about the advanced options you can use when you backup and restore a CockroachDB cluster.
toc: true
docs_area: manage
---

The ability to [backup a full cluster](backup.html#backup-a-cluster) has been added and the syntax for [incremental backups](backup.html#create-incremental-backups) is simplified. Because of these two changes, [basic backup usage](take-full-and-incremental-backups.html) is now sufficient for most CockroachDB clusters. However, you may want to control your backup and restore options more explicitly.

This doc provides information about how to take backups with revision history and restore from a point-in-time.

{{site.data.alerts.callout_info}}
[`BACKUP`](backup.html) with revision history is an [Enterprise-only](https://www.cockroachlabs.com/product/cockroachdb/) feature. However, you can take [full backups](take-full-and-incremental-backups.html) without an Enterprise license.
{{site.data.alerts.end}}

You can create full or incremental backups [with revision history](backup.html#with-revision-history):

- Taking full backups with revision history allows you to back up every change made within the garbage collection period leading up to and including the given timestamp.
- Taking incremental backups with revision history allows you to back up every change made since the last backup and within the garbage collection period leading up to and including the given timestamp. You can take incremental backups with revision history even when your previous full or incremental backups were taken without revision history.

You can configure garbage collection periods using the `ttlseconds` [replication zone setting](configure-replication-zones.html). Taking backups with revision history allows for point-in-time restores within the revision history.

## Create a backup with revision history

{% include_cached copy-clipboard.html %}
~~~ sql
> BACKUP INTO '{destination}' AS OF SYSTEM TIME '-10s' WITH revision_history;
~~~

For guidance on connecting to Amazon S3, Google Cloud Storage, Azure Storage, and other storage options, read [Use Cloud Storage for Bulk Operations](use-cloud-storage-for-bulk-operations.html).

## Point-in-time restore

If the full or incremental backup was taken [with revision history](#create-a-backup-with-revision-history), you can restore the data as it existed at an arbitrary point-in-time within the revision history captured by that backup. Use the [`AS OF SYSTEM TIME`](as-of-system-time.html) clause to specify the point-in-time.

Additionally, if you want to restore a specific incremental backup, you can do so by specifying the `end_time` of the backup by using the [`AS OF SYSTEM TIME`](as-of-system-time.html) clause. To find the incremental backup's `end_time`, use [`SHOW BACKUP`](show-backup.html).

If you do not specify a point-in-time, the data will be restored to the backup timestamp; that is, the restore will work as if the data was backed up without revision history.

{% include_cached copy-clipboard.html %}
~~~ sql
> RESTORE FROM '/2021/12/13-211056.62' IN '(destination)' AS OF SYSTEM TIME '2021-12-13 10:00:00';
~~~

To view the available backup subdirectories you can restore from, use [`SHOW BACKUPS`](restore.html#view-the-backup-subdirectories).

## See also

- [`BACKUP`][backup]
- [`RESTORE`][restore]
- [Take Full and Incremental Backups](take-full-and-incremental-backups.html)
- [Take and Restore Encrypted Backups](take-and-restore-encrypted-backups.html)
- [Take and Restore Locality-aware Backups](take-and-restore-locality-aware-backups.html)
- [`SQL DUMP`](cockroach-dump.html)
- [`IMPORT`](migration-overview.html)
- [Use the Built-in SQL Client](cockroach-sql.html)
- [Other Cockroach Commands](cockroach-commands.html)

<!-- Reference links -->

[backup]:  backup.html
[restore]: restore.html
