---
title: Upgrading from previous versions

menu:
  influxdb_012:
    weight: 30
    parent: administration
---

This page guides you through upgrading from InfluxDB 0.11 to 0.12.

> **Note:**
You can follow the same steps to upgrade from 0.10 to 0.12 (just replace all the
0.11 mentions with 0.10), but be sure to [convert](/influxdb/v0.10/administration/upgrading/#convert-b1-and-bz1-shards-to-tsm1)
any remaining `b1` and `bz1` shards to `TSM` format before you start. InfluxDB
0.12 cannot read non-`TSM` shards.
>
In your [data directory](/influxdb/v0.12/administration/config/#dir-var-lib-influxdb-data):
>
* Non-`TSM` shards are files of the form: `data/<database>/<retention_policy>/<shard_id>`
* `TSM` shards are files of the form: `data/<database>/<retention_policy>/<shard_id>/<file>.tsm`

In the next sections you will:

* Transfer your [metastore](/influxdb/v0.12/concepts/glossary/#metastore)
information to the new 0.12 store.
In versions prior to 0.12, InfluxDB stores metastore
information in `raft.db` via the raft services.
In 0.12, InfluxDB stores metastore information in `meta.db`, a binary protobuf
file.
* Generate a new configuration file.

To start out, you must be working with version 0.10 or 0.11 (don't upgrade the
`influxd` binary yet!).
If you've already upgraded the binary, reinstall 0.10 or 0.11; InfluxDB 0.12
will yield an error
(`run: create server: detected /var/lib/influxdb/meta/raft.db. [...]`) if you
attempt to start the process without completing the steps below.
The examples below assume you are working with a version of linux.

> Before you start, we recommend making a copy of the entire 0.11 `meta`
directory in case you experience problems with the upgrade. The upgrade process
removes the `raft.db` and `node.json` files from the `meta` directory:
>
```
cp -r <path_to_meta_directory> <path_to_011_meta_directory_backup>
```
>
Example:
>
Create a copy of the 0.11 `meta` directory in `backups/`:
```
~# cp -r /var/lib/influxdb/meta backups/
```

**1.** While still running 0.11, export the metastore data to a different
directory:

```
influxd backup <path_to_metastore_backup>
```

The directory will be created if it doesn't already exist.

Example:

Export the 0.11 metastore to `/tmp/backup`:
```
~# influxd backup /tmp/backup/
2016/04/01 15:33:35 backing up metastore to /tmp/backup/meta.00
2016/04/01 15:33:35 backup complete
```

**2.** Stop the `influxdb` service:

```
sudo service influxdb stop
```

**3.** [Upgrade](https://influxdata.com/downloads/#influxdb) the `influxd`
binary from 0.11 to 0.12. but do not start the service.

**4.** Upgrade your metastore to the 0.12 store by performing a `restore` with
the backup you created in step 1.

```
influxd restore -metadir=<path_to_012_meta_directory> <path_to_metastore_backup>
```

Example:

Restore `/tmp/backup` to the meta directory in `/var/lib/influxdb/meta`:
```
~# influxd restore -metadir=/var/lib/influxdb/meta /tmp/backup
Using metastore snapshot: /tmp/backup/meta.00
```

**5.** Generate a new configuration file.

InfluxDB 0.12 has several new settings in the [configuration file](/influxdb/v0.12/administration/config/).

The `influxd config` command prints out a new TOML-formatted configuration with all the available configuration options set to their default values.
On POSIX systems, a new configuration file can be generated by redirecting the output of the command to a file.

```
influxd config > /etc/influxdb/influxdb_012.conf.generated
```

Compare your old configuration file against the newly generated [InfluxDB 0.12 file](/influxdb/v0.12/administration/config/) and manually update any defaults with your localized settings.

**6.** Start the 0.12 service:

```
sudo service influxdb start
```

**7.** Confirm that your metastore data are present.

The 0.12 output from the queries `SHOW DATABASES`,`SHOW USERS` and
`SHOW RETENTION POLICIES ON <database_name>` should match the 0.11 output.

If your metastore data do not appear to be present, stop the service, reinstall
InfluxDB 0.11, restore the copy you made of the entire 0.11 `meta` directory to
the `meta` directory, and try working through these steps again.

**8.** Explore the new 0.12 features.

See [Differences between InfluxDB 0.12 and 0.11](/influxdb/v0.12/concepts/011_vs_012/).
