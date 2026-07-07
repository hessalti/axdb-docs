# Etcd setup

In our solutions, we use etcd distributed configuration store. [Refresh your knowledge about etcd](ha-components.md#database-and-dsc-layers).

## Install etcd

Use etcd under /opt/axdb/axdb-etcd/ on all PostgreSQL nodes: `node1`, `node2` and `node3`.

Create a dedicated system user for the `etcd` background process on every node:

```{.bash data-prompt="$"}
$ getent group etcd >/dev/null || sudo groupadd --system etcd
$ id -u etcd >/dev/null 2>&1 || sudo useradd --system --gid etcd --home-dir /var/lib/etcd --shell /sbin/nologin etcd
$ sudo mkdir -p /etc/etcd /var/lib/etcd
$ sudo chown -R etcd:etcd /etc/etcd /var/lib/etcd
```

This file allows `systemd` to start, stop, restart, and manage the `etcd` service. This includes handling dependencies, monitoring the service, and ensuring it runs as expected.

```ini title="/etc/systemd/system/etcd.service"
[Unit]
After=network.target
Description=etcd - highly-available key value store

[Service]
LimitNOFILE=65536
Restart=on-failure
Type=notify
ExecStart=/opt/axdb/axdb-etcd/bin/etcd --config-file /etc/etcd/etcd.conf.yaml
User=etcd
Group=etcd

[Install]
WantedBy=multi-user.target
```

## Configure etcd

To get started with `etcd` cluster, you need to bootstrap it. This means setting up the initial configuration and starting the etcd nodes so they can form a cluster. There are the following bootstrapping mechanisms:  

* Static in the case when the IP addresses of the cluster nodes are known
* Discovery service - for cases when the IP addresses of the cluster are not known ahead of time.
    
Since we know the IP addresses of the nodes, we will use the static method. For using the discovery service, please refer to the [etcd documentation :octicons-link-external-16:](https://etcd.io/docs/v3.5/op-guide/clustering/#etcd-discovery){:target="_blank"}.

We will configure and start all etcd nodes in parallel.

### Modify the configuration file

1. Create the etcd configuration file on every node. You can edit the sample configuration file `/etc/etcd/etcd.conf.yaml` or create your own one. Replace the node names and IP addresses with the actual names and IP addresses of your nodes. Make sure this file is owned by `etcd` user and group.

    === "node1"

         ```yaml title="/etc/etcd/etcd.conf.yaml"
         name: 'node1'
         initial-cluster-token: PostgreSQL_HA_Cluster_1
         initial-cluster-state: new
         initial-cluster: node1=http://192.168.3.201:2380,node2=http://192.168.3.202:2380,node3=http://192.168.3.203:2380
         data-dir: /var/lib/etcd
         initial-advertise-peer-urls: http://192.168.3.201:2380 
         listen-peer-urls: http://192.168.3.201:2380
         advertise-client-urls: http://192.168.3.201:2379
         listen-client-urls: http://192.168.3.201:2379
         ```

    === "node2"

         ```yaml title="/etc/etcd/etcd.conf.yaml"
         name: 'node2'
         initial-cluster-token: PostgreSQL_HA_Cluster_1
         initial-cluster-state: new
         initial-cluster: node1=http://192.168.3.201:2380,node2=http://192.168.3.202:2380,node3=http://192.168.3.203:2380
         data-dir: /var/lib/etcd
         initial-advertise-peer-urls: http://192.168.3.202:2380 
         listen-peer-urls: http://192.168.3.202:2380
         advertise-client-urls: http://192.168.3.202:2379
         listen-client-urls: http://192.168.3.202:2379
         ```

    === "node3"

         ```yaml title="/etc/etcd/etcd.conf.yaml"
         name: 'node3'
         initial-cluster-token: PostgreSQL_HA_Cluster_1
         initial-cluster-state: new
         initial-cluster: node1=http://192.168.3.201:2380,node2=http://192.168.3.202:2380,node3=http://192.168.3.203:2380
         data-dir: /var/lib/etcd
         initial-advertise-peer-urls: http://192.168.3.203:2380 
         listen-peer-urls: http://192.168.3.203:2380
         advertise-client-urls: http://192.168.3.203:2379
         listen-client-urls: http://192.168.3.203:2379
         ```

2. Enable and start the `etcd` service on all nodes:

    ```{.bash data-prompt="$"}
    $ sudo systemctl daemon-reload
    $ sudo systemctl enable etcd
    $ sudo systemctl start etcd
    $ sudo systemctl status etcd
    ```

    During the node start, etcd searches for other cluster nodes defined in the configuration. If the other nodes are not yet running, the start may fail by a quorum timeout. This is expected behavior. Try starting all nodes again at the same time for the etcd cluster to be created.

--8<-- "check-etcd.md"

## Next steps

[Patroni setup :material-arrow-right:](ha-patroni.md){.md-button}
