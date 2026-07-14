# Patroni setup

## Install Patroni

Use patroni under /opt/axdb/axdb-patroni/ (when `<axdb-dir>` is /opt/axdb/) on all PostgreSQL nodes: `node1`, `node2` and `node3`.
    
Stop and disable all installed patroni and postgresql services:
    
```{.bash data-prompt="$"}
$ sudo systemctl stop {patroni,postgresql}
$ sudo systemctl disable {patroni,postgresql}
```
    
Even though Patroni can use an existing Postgres instance, our recommendation for a **new cluster that has no data** is to have empty PostgreSQL data directory. This forces Patroni to initialize a new Postgres cluster instance.

**Don't** initialize the cluster and start the `postgresql`. The cluster initialization and setup are handled by Patroni during the bootsrapping stage.

## Configure Patroni

Run the following commands on all nodes. Patroni will run on postgres user. You can do this in parallel:

### Create environment variables 

Environment variables simplify the config file creation:

1. Node name:

    For example, run the following command for `node1`:

    ```{.bash data-prompt="$"}
    $ export NODE_NAME="node1"
    ```

2. Node IP:

    For example, run the following command for `node1`:

    ```{.bash data-prompt="$"}
    $ export NODE_IP="192.168.3.201"
    ```

3. Create variables to store the `PATH`. Check the path to the `data` and `bin` folders on your operating system and change it for the variables accordingly:

    === ":material-redhat: RHEL and derivatives"

    ```{.bash data-prompt="$"}
    $ export DATA_DIR="/usr/local/pgsql/data/"
    $ export PG_BIN_DIR="/opt/axdb/axdb-postgresql{{pgversion}}/bin"
    ```
    
4. Patroni information:

    ```{.bash data-prompt="$"}
    $ export NAMESPACE="axdb_lab"
    $ export SCOPE="cluster_1"       
    ```

### Create the directories required by Patroni

Create the directory to store the configuration file and make it owned by the `postgres` user.

```{.bash data-prompt="$"}
$ sudo mkdir -p /etc/patroni/
$ sudo chown -R  postgres:postgres /etc/patroni/
``` 

Create the directory to archive files and make it owned by the `postgres` user.

```{.bash data-prompt="$"}
$ sudo mkdir -p /home/postgres/archived/
$ sudo chown -R  postgres:postgres /home/postgres/archived/
``` 

While using a watchdog is optional, it is highly recommended. To enable it, verify that the watchdog device is functioning, uncomment the watchdog section in the configuration file, and ensure the `postgres` user has write permissions to it.

```{.bash data-prompt="$"}
$ sudo chown postgres:postgres /dev/watchdog
``` 

### Patroni configuration file

Use the following command to create the `/etc/patroni/patroni.yml` configuration file and add the following configuration for every node:

```bash
echo "
namespace: ${NAMESPACE}
scope: ${SCOPE}
name: ${NODE_NAME}

restapi:
  listen: 0.0.0.0:8008
  connect_address: ${NODE_IP}:8008

etcd3:
  hosts: 
    - 192.168.3.201:2379
    - 192.168.3.202:2379
    - 192.168.3.203:2379

bootstrap:
  # this section will be written into Etcd:/<namespace>/<scope>/config after initializing new cluster
  dcs:
    ttl: 30
    loop_wait: 10
    retry_timeout: 10
    maximum_lag_on_failover: 1048576

    postgresql:
      use_pg_rewind: true
      use_slots: true
      parameters:
        wal_level: replica
        hot_standby: on
        max_wal_senders: 5
        max_replication_slots: 10
        max_slot_wal_keep_size: 10GB
        wal_keep_size: 256MB
        max_connections: 200
        wal_log_hints: on
        logging_collector: on
        max_wal_size: 10GB
        archive_mode: on
        archive_timeout: 600
        archive_command: 'cp -f %p /home/postgres/archived/%f'
      recovery_conf:
        restore_command: 'cp /home/postgres/archived/%f %p'
        recovery_target_timeline: latest
    
  # some desired options for 'initdb'
  initdb: # Note: It needs to be a list (some options need values, others are switches)
    - encoding: UTF8
    - data-checksums

  ####  - host replication replicator 192.168.3.0/24 trust
  pg_hba: # Add following lines to pg_hba.conf after running 'initdb'
    - host replication replicator 127.0.0.1/32 trust
    - host replication replicator 0.0.0.0/0 md5
    - host all all 0.0.0.0/0 md5
    - host all all ::0/0 md5
    
postgresql:
  cluster_name: cluster_1
  listen: 0.0.0.0:5432
  connect_address: ${NODE_IP}:5432
  data_dir: ${DATA_DIR}
  bin_dir: ${PG_BIN_DIR}
  pgpass: /tmp/pgpass
  authentication:
    replication:
      username: replicator
      password: replPasswd
    superuser:
      username: postgres
      password: qaz123
  parameters:
    unix_socket_directories: "/tmp"
  create_replica_methods:
    - basebackup
  basebackup:
    checkpoint: 'fast'

#watchdog:
#  mode: required # Allowed values: off, automatic, required
#  device: /dev/watchdog
#  safety_margin: 5

tags:
  nofailover: false
  noloadbalance: false
  clonefrom: false
  nosync: false
" | tee /etc/patroni/patroni.yml
```

??? admonition "Patroni configuration file"

    Let’s take a moment to understand the contents of the `patroni.yml` file. 

    The first section provides the details of the node and its connection ports. After that, we have the `etcd` service and its port details.

    Following these, there is a `bootstrap` section that contains the PostgreSQL configurations and the steps to run once.

    Since these are plain-text passwords stored in a configuration file, ensure your patroni.yaml file is securely locked down so that only the postgres user can read it (e.g., chmod 600 patroni.yaml).

### Systemd configuration

1. Check that the systemd unit file `patroni.service` is created in `/etc/systemd/system`. If it is created, skip this step. 

    If it's **not created**, create it manually and specify the following contents within:

    ```ini title="/etc/systemd/system/patroni.service"
    [Unit]
    Description=Runners to orchestrate a high-availability PostgreSQL
    After=syslog.target network.target 

    [Service]
    Type=simple 

    User=postgres
    Group=postgres 

    # Start the patroni process
    ExecStart=/opt/axdb/axdb-patroni/bin/patroni /etc/patroni/patroni.yml 

    # Send HUP to reload from patroni.yml
    ExecReload=/bin/kill -s HUP $MAINPID 

    # only kill the patroni process, not its children, so it will gracefully stop postgres
    KillMode=process 

    # Give a reasonable amount of time for the server to start up/shut down
    TimeoutSec=30 

    # Do not restart the service if it crashes, we want to manually inspect database on failure
    Restart=no 

    [Install]
    WantedBy=multi-user.target
    ```

2. Make `systemd` aware of the new service:

    ```{.bash data-prompt="$"}
    $ sudo systemctl daemon-reload
    ```

3. Make sure you have the configuration file and the `systemd` unit file created on every node. 

### Start Patroni

Now it's time to start Patroni. You need the following commands on all nodes but **not in parallel**. 

1. Start Patroni on the primary node(in this example, `node1`) first, wait for the service to come to live, and then proceed with the other nodes one-by-one, always waiting for them to sync with the primary node:

    ```{.bash data-prompt="$"}
    $ sudo systemctl enable patroni
    $ sudo systemctl start patroni
    $ sudo systemctl status patroni
    ```

    When Patroni starts, it initializes PostgreSQL (because the service is not currently running and the data directory is empty) following the directives in the bootstrap section of the configuration file. 

2. Check the service to see if there are errors:

    ```{.bash data-prompt="$"}
    $ sudo journalctl -fu patroni
    ```

    See [Troubleshooting Patroni startup](#troubleshooting-patroni-startup) for guidelines in case of errors. 

    If Patroni has started properly, you should be able to locally connect to a PostgreSQL node using the following command:

    ```{.bash data-prompt="$"}
    $ psql -U postgres

    psql ({{dockertag}})
    Type "help" for help.

    postgres=#
    ```

9. When all nodes are up and running, you can check the cluster status using the following command:

    ```{.bash data-prompt="$"}
    $ patronictl -c /etc/patroni/patroni.yml list
    ```
    
    The output resembles the following:

    ??? example "Sample output"

        ```{.text .no-copy}
        + Cluster: cluster_1 (7662233048390239093) ----+----+-------------+-----+------------+-----+
        | Member | Host          | Role    | State     | TL | Receive LSN | Lag | Replay LSN | Lag |
        +--------+---------------+---------+-----------+----+-------------+-----+------------+-----+
        | node1  | 192.168.3.201 | Leader  | running   |  1 |             |     |            |     |
        | node2  | 192.168.3.202 | Replica | streaming |  1 |   0/6000000 |   0 |  0/6000000 |   0 |
        | node3  | 192.168.3.203 | Replica | streaming |  1 |   0/6000000 |   0 |  0/6000000 |   0 |
        +--------+---------------+---------+-----------+----+-------------+-----+------------+-----+
        ```

### Troubleshooting Patroni startup

 A common error is Patroni complaining about the lack of proper entries in the `pg_hba.conf` file. If you see such errors, you must manually add or fix the entries in that file and then restart the service.

An example of such an error is `No pg_hba.conf entry for replication connection from host to <IP>, user replicator, no encryption`. This means that Patroni cannot connect to the node you're adding to the cluster. To resolve this issue, add the IP addresses of the nodes to the `pg_hba:` section of the Patroni configuration file. 

```
pg_hba: # Add following lines to pg_hba.conf after running 'initdb'
- host replication replicator 127.0.0.1/32 trust
- host replication replicator 0.0.0.0/0 md5
- host replication replicator 192.168.3.202/32 trust
- host replication replicator 192.168.3.203/32 trust
- host all all 0.0.0.0/0 md5
- host all all ::0/0 md5
```

For production use, we recommend adding nodes individually as the more secure way. However, if your network is secure and you trust it, you can add the whole network these nodes belong to as the trusted one to bypass passwords use during authentication. Then all nodes from this network can connect to Patroni cluster. 

Changing the `patroni.yml` file and restarting the service will not have any effect here because the bootstrap section specifies the configuration to apply when PostgreSQL is first started in the node. It will not repeat the process even if the Patroni configuration file is modified and the service is restarted. 

## Next steps

[pgBackRest setup :material-arrow-right:](pgbackrest.md){.md-button}
