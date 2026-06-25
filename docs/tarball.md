# Install AXDB from binary tarballs

You can download the tarballs using the links below.

The following tarballs are available for the x86_64 architectures:

* axdb-ssl3.5-linux-x86_64.tar.gz - for operating systems on x86_64 architecture that run OpenSSL version 3.5.x

To check what OpenSSL version you have, run the following command:

```{.bash data-prompt="$"}
$ openssl version
```

## Tarball contents

The tarballs include the following components:

| Component | Description |
|-----------|-------------|
| axdb{{pgversion}}| The latest version of PostgreSQL server and the following extensions: <br> - `pgaudit` <br> - `pgAudit_set_user` <br> - `pg_repack` <br> - `pg_stat_monitor` <br> - `pg_gather` <br> - `wal2json` <br> - `postGIS` <br> -  the set of [contrib extensions](contrib.md)|
| percona-haproxy | A high-availability solution and load-balancing solution |
| percona-patroni | A high-availability solution for PostgreSQL |
| percona-pgbackrest| A backup and restore tool |
| percona-pgbadger| PostgreSQL log analyzer with fully detailed reports and graphs |
| percona-pgbouncer| Lightweight connection pooler for PostgreSQL |
| percona-pgpool-II| A middleware between PostgreSQL server and client for high availability, connection pooling and load balancing |
| percona-perl | A Perl module required to create the `plperl` extension - a procedural language handler for PostgreSQL that allows writing functions in the Perl programming language|
| percona-python3 | A Python3 module required to create `plpython` extension - a procedural language handler for PostgreSQL that allows writing functions in the Python programming language. Python is also required by Patroni
| percona-tcl | Tcl development libraries required to create the `pltcl` extension - a loadable procedural language for the PostgreSQL database system that enables the creation of functions and trigger procedures in the Tcl language |
| percona-etcd | A key-value distributed store that stores the state of the PostgreSQL cluster|

## Preconditions

=== "Debian and Ubuntu"

    1. Uninstall the upstream PostgreSQL package.
    2. Ensure that the `libreadline` is installed on the system, as it is **required** for tarballs to work correctly:

        ```{.bash data-prompt="$"}
        $ sudo apt install -y libreadline-dev
        ```

    3. Create the user to own the PostgreSQL process. For example, `mypguser`. Run the following command:

        ```{.bash data-prompt="$"}
        $ sudo useradd -m mypguser
        ```

        Set the password for the user:

        ```{.bash data-prompt="$"}
        $ sudo passwd mypguser
        ```
    
=== "RHEL and derivatives"

    On RHEL, Rocky Linux, or Oracle Linux 10, install the `acl` package. This package is **required** for correct permission handling when using tarball-based installations:

    ```{.bash data-prompt="$"}
    $ sudo dnf install -y acl
    ```

    Ensure that the `libreadline` is installed on the system, as it is **required** for tarballs to work correctly:

    ```{.bash data-prompt="$"}
    $ sudo yum install -y readline-devel
    ```

    Create the user to own the PostgreSQL process. For example, `mypguser`. Run the following command:
        
    ```{.bash data-prompt="$"}
    $ sudo useradd mypguser -m 
    ```

    Set the password for the user:

    ```{.bash data-prompt="$"}
    $ sudo passwd mypguser
    ```

## Procedure

The steps below install the tarballs for OpenSSL 3.x on x86_64 architecture. Use another tarball if your operating system has OpenSSL version 1.x and / or has the ARM64 architecture.

1. Create the directory where you will store the binaries. For example, `/opt/pgdistro`

2. Fetch the binary tarball.

    ```{.bash data-prompt="$"}
    $ wget https://downloads.altibase.com/downloads/postgresql-distribution-{{pgversion}}/{{dockertag}}/binary/tarball/axdb-{{dockertag}}-ssl3-linux-x86_64.tar.gz
    ```

3. Extract the tarball to the directory for binaries that you created on step 1.

    ```{.bash data-prompt="$"}
    $ sudo tar -xvf axdb-{{dockertag}}-ssl3-linux-x86_64.tar.gz -C /opt/pgdistro/
    ```

5. Copy `percona-python3`, `percona-tcl` and `percona-perl` to the `/opt` directory. This is required for the correct run of libraries that require those modules.

    ```{.bash data-prompt="$"}
    $ sudo cp -r /opt/percona-perl /opt/percona-python3 /opt/percona-tcl /opt/
    ```

6. Add the location of the binaries to the PATH variable:

    ```{.bash data-prompt="$"}
    $ export PATH=:/opt/pgdistro/percona-haproxy/sbin/:/opt/pgdistro/percona-patroni/bin/:/opt/pgdistro/percona-pgbackrest/bin/:/opt/pgdistro/percona-pgbadger/:/opt/pgdistro/percona-pgbouncer/bin/:/opt/pgdistro/percona-pgpool-II/bin/:/opt/pgdistro/percona-postgresql{{pgversion}}/bin/:/opt/pgdistro/percona-etcd/bin/:/opt/percona-perl/bin/:/opt/percona-tcl/bin/:/opt/percona-python3/bin/:$PATH
    ```

6. Create the data directory for PostgreSQL server. For example, `/usr/local/pgsql/data`.
7. Grant access to these directory for the `mypguser` user.

    ```{.bash data-prompt="$"}
    $ sudo chown -R mypguser:mypguser /opt/axdb/
    $ sudo chown -R mypguser:mypguser /opt/axdb-perl/
    $ sudo chown -R mypguser:mypguser /opt/axdb-python3/
    $ sudo chown -R mypguser:mypguser /opt/axdb-tcl/
    $ sudo chown mypguser:mypguser /usr/local/pgsql/data
    ```

8. Switch to the user that owns the Postgres process. In our example, `mypguser`:

    ```{.bash data-prompt="$"}
    $ su - mypguser
    ```

9. Initiate the PostgreSQL data directory:

    ```{.bash data-prompt="$"}
    $ /opt/pgdistro/axdb{{pgversion}}/bin/initdb -D /usr/local/pgsql/data
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        Success. You can now start the database server using:

        /opt/pgdistro/axdb{{pgversion}}/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
        ```

10. Start the PostgreSQL server:

    ```{.bash data-prompt="$"}
    $ /opt/pgdistro/axdb{{pgversion}}/bin/pg_ctl -D /usr/local/pgsql/data -l logfile start
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        waiting for server to start.... done
        server started
        ```

9. Connect to `psql`

    ```{.bash data-prompt="$"}
    $ /opt/pgdistro/axdb{{pgversion}}/bin/psql -d postgres
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        psql ({{pspgversion}} (AXDB), server {{pspgversion}} (AXDB))
        Type "help" for help.

        postgres=#
        ```

### Start the components

After you unpacked the tarball and added the location of the components' binaries to the `$PATH` variable, the components are available for use. You can invoke a component by running its command-line tool.

For example, to check HAProxy version, type:

```{.bash data-prompt="$"}
$ haproxy -v
```

Some components require additional setup. Check the [Enabling extensions](enable-extensions.md) page for details.
