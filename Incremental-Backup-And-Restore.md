# Incremental Backup and Restore

(Since v2.2)

Incremental backup generates smaller backup files with only the delta between 2 versions of databases. This is useful when you execute a backup at regular basis and you want to avoid to backup the entire database everytime.

## How does it work?

Every time a backup is executed, OrientDB writes in the database directory, a file named `last-backup.json`. This is an example of the content:

```json
{
  "lsn": 8438432,
  "startedOn": "2015-08-17 10:33:23.943",
  "completedOn": "2015-08-17 10:33:45.121"
}
```

The most important information is the `lsn` field that is the WAL LSN (Last Serial Number). Thanks to this number, OrientDB is able to understand the last change in database, so the next incremental backup will be done starting from last `lsn` + 1.

## Execute an incremental backup

### Incremental backup via console

[Backup Database console command](Console-Command-Backup-Database.md) accepts `-incremental` as optional parameter to execute an incremental backup. In this case the new backup is executed from last backup executed (File `last-backup.json` is read if any). If this is the first incremental backup, a full backup is executed. Example:

```
orientdb {db=Whisky}> backup /tmp/backup.zip -incremental
```

Incremental backup setting allows also to specify a specific version (LSN) to start. Example:

```
orientdb {db=Whisky}> backup /tmp/backup.zip -incremental=93222
```



## Distributed Architecture

Incremental backup is used in [Distributed Architecture](Distributed-Architecture.md) when a server node restarts. This avoid to backup and tranfer the entire database across the network.

## Internals

### File format
In case of incremental backup, the content of the zip file is not the database directory, but rather meta files needed to update the database with the delta. Example of content:

```
- Person.pcl
- Person.pcm
```

This means only 2 files are changed. Internally the file contains all the page changes with the following format:
```
+----------------+-----------------+
| PAGE NUMBER    | PAGE CONTENT    |
| (long)         | byte[]          |
+----------------+-----------------+
```