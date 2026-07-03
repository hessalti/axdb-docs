# Install AXDB from binary tarballs

You can download the tarballs using the links below.

The following tarballs are available for the x86_64 architectures:

* [axdb-enterprise-release-{{dockertag}}-ssl3.5-linux-x86_64.tar.gz](https://drive.google.com/file/d/1CCgdR631PiTiu3XqN1GTWeqVaz1ovuZ4/view?usp=sharing) - for operating systems on x86_64 architecture that run OpenSSL version 3.5.x

To check what OpenSSL version you have, run the following command:

```{.bash data-prompt="$"}
$ openssl version
```

## Tarball contents

The tarballs include the following components:

| Component | Description |
|-----------|-------------|
| axdb-postgresql{{pgversion}}| The latest version of PostgreSQL server and the following extensions: <br> - `pgaudit` <br> - `pgAudit_set_user` <br> - `pg_repack` <br> - `pg_stat_monitor` <br> - `pg_gather` <br> - `wal2json` <br> - `postGIS` <br> -  the set of [contrib extensions](contrib.md)|
| axdb-haproxy | A high-availability solution and load-balancing solution |
| axdb-patroni | A high-availability solution for PostgreSQL |
| axdb-pgbackrest| A backup and restore tool |
| axdb-pgbadger| PostgreSQL log analyzer with fully detailed reports and graphs |
| axdb-pgbouncer| Lightweight connection pooler for PostgreSQL |
| axdb-pgpool-II| A middleware between PostgreSQL server and client for high availability, connection pooling and load balancing |
| axdb-perl | A Perl module required to create the `plperl` extension - a procedural language handler for PostgreSQL that allows writing functions in the Perl programming language|
| axdb-python3 | A Python3 module required to create `plpython` extension - a procedural language handler for PostgreSQL that allows writing functions in the Python programming language. Python is also required by Patroni
| axdb-tcl | Tcl development libraries required to create the `pltcl` extension - a loadable procedural language for the PostgreSQL database system that enables the creation of functions and trigger procedures in the Tcl language |
| axdb-etcd | A key-value distributed store that stores the state of the PostgreSQL cluster|

## Preconditions

=== "RHEL and derivatives"

    On RHEL, Rocky Linux, or Oracle Linux 10, install the `acl` package. This package is **required** for correct permission handling when using tarball-based installations:

    ```{.bash data-prompt="$"}
    $ sudo dnf install -y acl
    ```

    Ensure that the `libreadline` is installed on the system, as it is **required** for tarballs to work correctly:

    ```{.bash data-prompt="$"}
    $ sudo dnf install -y readline-devel
    ```

    Create the user to own the PostgreSQL process. For example, `postgres`. Run the following command:
        
    ```{.bash data-prompt="$"}
    $ sudo useradd postgres -m 
    ```

    Set the password for the user:

    ```{.bash data-prompt="$"}
    $ sudo passwd postgres
    ```

## Procedure

The steps below install the tarballs for OpenSSL 3.5.x on x86_64 architecture.

1. Create the directory where you will store the binaries. For example, `/opt/axdb`

2. Get the binary tarball. [axdb-enterprise-release-{{dockertag}}-ssl3.5-linux-x86_64.tar.gz](https://drive.google.com/file/d/1CCgdR631PiTiu3XqN1GTWeqVaz1ovuZ4/view?usp=sharing)

3. Extract the tarball to the directory for binaries that you created on step 1.

    ```{.bash data-prompt="$"}
    $ sudo tar -xvf axdb-enterprise-release-{{dockertag}}-ssl3.5-linux-x86_64.tar.gz -C /opt/axdb/
    ```

4. Copy `axdb-python3`, `axdb-tcl` and `axdb-perl` to the `/opt` directory. This is required for the correct run of libraries that require those modules.

    ```{.bash data-prompt="$"}
    $ sudo cp -r /opt/axdb-perl /opt/axdb-python3 /opt/axdb-tcl /opt/
    ```

5. Create the data directory for PostgreSQL server. For example, `/usr/local/pgsql/data`.
6. Grant access to these directory for the `postgres` user.

    ```{.bash data-prompt="$"}
    $ sudo chown -R postgres:postgres /opt/axdb/
    $ sudo chown -R postgres:postgres /opt/axdb-perl/
    $ sudo chown -R postgres:postgres /opt/axdb-python3/
    $ sudo chown -R postgres:postgres /opt/axdb-tcl/
    $ sudo chown -R postgres:postgres /usr/local/pgsql/data
    ```

7. Switch to the user that owns the Postgres process. In our example, `postgres`:

    ```{.bash data-prompt="$"}
    $ su - postgres
    ```

8. Add the location of the binaries to the PATH variable:

    ```{.bash data-prompt="$"}
    export PATH=:/opt/axdb/axdb-haproxy/sbin/:/opt/axdb/axdb-patroni/bin/:/opt/axdb/axdb-pgbackrest/bin/:/opt/axdb/axdb-pgbadger/:/opt/axdb/axdb-pgbouncer/bin/:/opt/axdb/axdb-pgpool-II/bin/:/opt/axdb/axdb-postgresql{{pgversion}}/bin/:/opt/axdb/axdb-etcd/bin/:/opt/axdb-perl/bin/:/opt/axdb-tcl/bin/:/opt/axdb-python3/bin/:$PATH;
    export LD_LIBRARY_PATH=/opt/axdb/axdb-postgresql{{pgversion}}/lib:$LD_LIBRARY_PATH;
    ```

9. Initiate the PostgreSQL data directory:

    ```{.bash data-prompt="$"}
    $ /opt/axdb/axdb-postgresql{{pgversion}}/bin/initdb -D /usr/local/pgsql/data
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        Success. You can now start the database server using:

        /opt/axdb/axdb-postgresql{{pgversion}}/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
        ```

10. Start the PostgreSQL server:

    ```{.bash data-prompt="$"}
    $ /opt/axdb/axdb-postgresql{{pgversion}}/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        waiting for server to start.... done
        server started
        ```

11. Connect to `psql`

    ```{.bash data-prompt="$"}
    $ /opt/axdb/axdb-postgresql{{pgversion}}/bin/psql -d postgres
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        psql ({{dockertag}} AXDB Server for PostgreSQL {{pspgversion}})
        Type "help" for help.

	        postgres=#
	        ```

### Configure PostgreSQL as a systemd service

Because you installed PostgreSQL from a tarball, `systemd` doesn't automatically know it exists. To register it as a system service so it starts automatically on boot and responds to `systemctl` commands, manually create a systemd service unit file.

#### Prerequisites

Before starting, ensure you know two paths from your tarball installation:

1. The binary directory where `pg_ctl` lives. For example, `/opt/axdb/axdb-postgresql{{pgversion}}/bin/`.
2. The data directory where your database cluster is initialized. For example, `/usr/local/pgsql/data/`.

For security reasons, PostgreSQL should never run as `root`. Ensure that the system user who owns the PostgreSQL process also owns the data directory. In this procedure, the example user is `postgres`.

#### Step 1: Create the systemd service file

Open a new file inside `/etc/systemd/system/` using your preferred text editor. This requires `sudo`:

```{.bash data-prompt="$"}
$ sudo nano /etc/systemd/system/postgresql.service
```

Paste the following configuration into the file. Update the paths if your tarball was extracted somewhere else:

```ini
[Unit]
Description=PostgreSQL database server (tarball installation)
After=network.target

[Service]
Type=forking
User=postgres
Group=postgres

# Path to your database storage cluster
Environment=PGDATA=/usr/local/pgsql/data
Environment="LD_LIBRARY_PATH=/opt/axdb/axdb-postgresql{{pgversion}}/lib:$LD_LIBRARY_PATH;"

# Start, stop, and reload PostgreSQL using pg_ctl
ExecStart=/opt/axdb/axdb-postgresql{{pgversion}}/bin/pg_ctl start -D ${PGDATA} -s
ExecStop=/opt/axdb/axdb-postgresql{{pgversion}}/bin/pg_ctl stop -D ${PGDATA} -s -m fast
ExecReload=/opt/axdb/axdb-postgresql{{pgversion}}/bin/pg_ctl reload -D ${PGDATA} -s

# Give the server up to 10 minutes to start up or shut down safely
TimeoutSec=600

[Install]
WantedBy=multi-user.target
```

Save and exit the file.

#### Step 2: Reload systemd and enable the service

After the file is in place, refresh the systemd manager configuration so it recognizes the new service:

```{.bash data-prompt="$"}
$ sudo systemctl daemon-reload
```

Enable the service so it starts automatically when the machine boots:

```{.bash data-prompt="$"}
$ sudo systemctl enable postgresql
```

#### Step 3: Manage the service

You can now control your PostgreSQL installation with `systemctl`.

If PostgreSQL is currently running via a manual start, you must stop it manually before switching to systemctl.

To start PostgreSQL, run:

```{.bash data-prompt="$"}
$ sudo systemctl start postgresql
```

To check the service status, run:

```{.bash data-prompt="$"}
$ sudo systemctl status postgresql
```

To stop PostgreSQL, run:

```{.bash data-prompt="$"}
$ sudo systemctl stop postgresql
```

To reload the PostgreSQL configuration, run:

```{.bash data-prompt="$"}
$ sudo systemctl reload postgresql
```

!!! tip "Troubleshooting"

    If `systemctl status postgresql` reports an error, check the data directory permissions. Make sure the `postgres` user owns the `PGDATA` directory:

    ```{.bash data-prompt="$"}
    $ sudo chown -R postgres:postgres /usr/local/pgsql/data
    ```

### Start the components

After you unpacked the tarball and added the location of the components' binaries to the `$PATH` variable, the components are available for use. You can invoke a component by running its command-line tool.

For example, to check HAProxy version, type:

```{.bash data-prompt="$"}
$ haproxy -v
```

Some components require additional setup. Check the [Enabling extensions](enable-extensions.md) page for details.
