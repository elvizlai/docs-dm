---
title: Deploy Data Migration Using DM Binary
summary: Learn how to deploy a Data Migration cluster using DM binary.
category: how-to
aliases: ['/docs/tidb-data-migration/dev/deploy-a-dm-cluster-using-binary/']
---

# Deploy Data Migration Using DM Binary

This document introduces how to quickly deploy the Data Migration (DM) cluster using DM binary.

## Preparations

Download the official binary from [here](https://pingcap.com/docs/dev/reference/tools/download/#tidb-dm-data-migration).

The downloaded files have two subdirectories, `bin` and `conf`. The `bin` directory contains the binary files of DM-master, DM-worker, and dmctl. The `conf` directory contains the sample configuration files.

## Sample scenario

Suppose that you deploy a DM cluster based on this sample scenario:

Two DM-worker nodes and three DM-master nodes are deployed on five servers.

Here is the address of each node:

| Instance | Server address | Port |
| :---------- | :----------- | :-- |
| DM-master1 | 192.168.0.4 | 8261 |
| DM-master2 | 192.168.0.5 | 8261 |
| DM-master3 | 192.168.0.6 | 8261 |
| DM-worker1 | 192.168.0.7 | 8262 |
| DM-worker2 | 192.168.0.8 | 8262 |

Based on this scenario, the following sections describe how to deploy the DM cluster.

### Deploy DM-master

You can configure DM-master by using [command-line parameters](#dm-master-command-line-parameters) or [the configuration file](#dm-master-configuration-file).

#### DM-master command-line parameters

The following is the description of DM-master command-line parameters:

```bash
./bin/dm-master --help
```

```
Usage of dm-master:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${master-addr}")
  -advertise-peer-urls string
        advertise URLs for peer traffic (default "${peer-urls}")
  -config string
        path to config file
  -data-dir string
        path to the data directory (default "default.${name}")
  -initial-cluster string
        initial cluster configuration for bootstrapping, e.g. dm-master=http://127.0.0.1:8291
  -join string
        join to an existing cluster (usage: cluster's "${master-addr}" list, e.g. "127.0.0.1:8261,127.0.0.1:18261"
  -log-file string
        log file path
  -master-addr string
        master API server and status addr
  -name string
        human-readable name for this DM-master member
  -peer-urls string
        URLs for peer traffic (default "http://127.0.0.1:8291")
  -print-sample-config
        print sample config file of dm-worker
```

> **Note:**
>
> In some situations, you cannot use the above method to configure DM-master because some configurations are not exposed to the command line. In such cases, use the configuration file instead.

#### DM-master configuration file

The following is the configuration file of DM-master. It is recommended that you configure DM-master by using this method.

1. Write the following configuration to `conf/dm-master1.toml`:

      ```toml
      # Master Configuration.
      name = "master1"

      # Log configurations.
      log-level = "info"
      log-file = "dm-master.log"

      # The listening address of DM-master.
      master-addr = ":8261"

      # The peer URLs of DM-master.
      peer-urls = "192.168.0.4:8291"

      # The value of `initial-cluster` is the combination of the `advertise-peer-urls` value of all DM-master nodes in the initial cluster.
      initial-cluster = "master1=http://192.168.0.4:8291,master2=http://192.168.0.5:8291,master3=http://192.168.0.6:8291"
      ```

2. Execute the following command in the terminal to run DM-master:

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dm-master -config conf/dm-master1.toml
      ```

3. For DM-master2 and DM-master3, change `name` in the configuration file to `master2` and `master3` respectively, and change `peer-urls` to `192.168.0.5:8291` and `192.168.0.6:8291` respectively. Then repeat Step 2.

### Deploy DM-worker

You can configure DM-worker by using [command-line parameters](#dm-worker-command-line-parameters) or [the configuration file](#dm-worker-configuration-file).

#### DM-worker command-line parameters

The following is the description of the DM-worker command-line parameters:

{{< copyable "shell-regular" >}}

```bash
./bin/dm-worker --help
```

```
Usage of worker:
  -L string
        log level: debug, info, warn, error, fatal (default "info")
  -V    prints version and exit
  -advertise-addr string
        advertise address for client traffic (default "${worker-addr}")
  -config string
        path to config file
  -join string
        join to an existing cluster (usage: dm-master cluster's "${master-addr}")
  -keepalive-ttl int
        dm-worker's TTL for keepalive with etcd (in seconds) (default 10)
  -log-file string
        log file path
  -name string
        human-readable name for DM-worker member
  -print-sample-config
        print sample config file of dm-worker
  -worker-addr string
        listen address for client traffic
```

> **Note:**
>
> In some situations, you cannot use the above method to configure DM-worker because some configurations are not exposed to the command line. In such cases, use the configuration file instead.

#### DM-worker configuration file

The following is the DM-worker configuration file. It is recommended that you configure DM-worker by using this method.

1. Write the following configuration to `conf/dm-worker1.toml`:

      ```toml
      # Worker Configuration.
      name = "worker1"

      # Log configuration.
      log-level = "info"
      log-file = "dm-worker.log"

      # DM-worker address.
      worker-addr = ":8262"

      # The master-addr configuration of the DM-master nodes in the cluster.
      join = "192.168.0.4:8261,192.168.0.5:8261,192.168.0.6:8261"
      ```

2. Execute the following command in the terminal to run DM-worker:

      {{< copyable "shell-regular" >}}

      ```bash
      ./bin/dm-worker -config conf/dm-worker1.toml
      ```

3. For DM-worker2, change `name` in the configuration file to `worker2`. Then repeat Step 2.

Now, a DM cluster is successfully deployed.
