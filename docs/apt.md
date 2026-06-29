# Install AXDB on Debian and Ubuntu

This document describes how to install AXDB Server for PostgreSQL from AXDB repositories on DEB-based distributions such as Debian and Ubuntu. [Read more about AXDB repositories](repo-overview.md).

## Preconditions

Debian and other systems that use the `apt` package manager include the upstream PostgreSQL server package `postgresql-{{pgversion}}` by default.

The components of AXDB {{pgversion}} can only be installed together with AXDB Server for PostgreSQL (`axdb-{{pgversion}}`).

If you wish to use AXDB, uninstall the `postgresql-{{pgversion}}` package provided by your distribution and then install the chosen components from AXDB.

## Procedure

Run all the commands in the following sections as root or using the `sudo` command:

### Configure AXDB repository {.power-number}

1. Install the `axdb-release` repository management tool to subscribe to AXDB repositories:

     * Fetch `axdb-release` packages from AXDB web:

        ```{.bash data-prompt="$"}
        $ wget https://repo.axdb.com/apt/axdb-release_latest.$(lsb_release -sc)_all.deb
        ```

     * Install the downloaded package with `dpkg`:

        ```{.bash data-prompt="$"}
        $ sudo dpkg -i axdb-release_latest.$(lsb_release -sc)_all.deb
        ```

     * Refresh the local cache:

        ```{.bash data-prompt="$"}
        $ sudo apt update
        ```

2. Enable the repository

   AXDB provides [two repositories](repo-overview.md) for AXDB. We recommend enabling the Major release repository to timely receive the latest updates. 

   ```{.bash data-prompt="$"}
   $ sudo axdb-release setup axdb-{{pgversion}}
   ```

### Install packages individually

To install the packages individually, run the following commands:
{.power-number}

1. Install the PostgreSQL server package:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-{{pgversion}}
    ```

2. Install the components:

    Install `pg_repack`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-{{pgversion}}-repack
    ```

    Install `pgAudit`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-{{pgversion}}-pgaudit
    ```

    Install `pgBackRest`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pgbackrest
    ```

    Install `Patroni`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-patroni
    ```

    [Install `pg_stat_monitor` :octicons-link-external-16:](https://docs.percona.com/pg-stat-monitor/install.html#__tabbed_1_1).

    Install `pgBouncer`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pgbouncer
    ```

    Install `pgAudit-set_user`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pgaudit{{pgversion}}-set-user
    ```

    Install `pgBadger`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pgbadger
    ```

    Install `wal2json`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-{{pgversion}}-wal2json
    ```

    Install `PostgreSQL contrib` extensions:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-contrib
    ```

    Install `HAProxy`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-haproxy
    ```

    Install `pgpool2`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pgpool2
    ```

    Install `pg_gather`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-pg-gather
    ```

    Install `pgvector`:

    ```{.bash data-prompt="$"}
    $ sudo apt install axdb-{{pgversion}}-pgvector
    ```

    Some extensions require additional setup in order to use them with AXDB. For more information, refer to [Enabling extensions](enable-extensions.md).

### Start the service

The installation process automatically initializes and starts the default database. You can check the database status using the following command:

```{.bash data-prompt="$"}
$ sudo systemctl status postgresql.service
```

Check the AXDB version:

```{.bash data-prompt="$"}
$ psql --version
```

??? example "Sample output"

    ```{.text .no-copy}
    psql (PostgreSQL) {{pspgversion}} (AXDB Server for PostgreSQL) {{pspgversion}}
    ```

Congratulations! Your AXDB is up and running.

## Next steps

[Enable extensions :material-arrow-right:](enable-extensions.md){.md-button}

[Connect to PostgreSQL :material-arrow-right:](connect.md){.md-button}
