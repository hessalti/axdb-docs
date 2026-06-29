# Minor Upgrade of AXDB

Minor releases of PostgreSQL include bug fixes and feature enhancements. We recommend that you keep your AXDB updated to the latest minor version.

Though minor upgrades do not change the behavior, we recommend you to back up your data first, in order to be on the safe side.

!!! note

    These steps apply if you installed AXDB from the Major Release repository. In this case, you are always upgraded to the latest available release.

    If you installed AXDB from the Minor Release repository, you will need to enable a new version repository to upgrade.

    For more information about AXDB repositories, refer to [Installing AXDB](installing.md).

## Before you start

1. [Update the `axdb-release` :octicons-link-external-16:](https://hessalti.github.io/repo-config-docs/latest/updating.html) utility to the latest version. This is required to install the new version packages of AXDB.

## Procedure

Run **all** commands as root or via **sudo**:
{.power-number}

1. Stop the `postgresql` service:

    === ":material-debian: On Debian / Ubuntu"

         ```{.bash data-prompt="$"}
         $ sudo systemctl stop postgresql.service
         ```

    === ":material-redhat: On Red Hat Enterprise Linux / derivatives"

         ```{.bash data-prompt="$"}
         $ sudo systemctl stop postgresql-18
         ```

2. [Update `axdb-release` to the latest version](https://hessalti.github.io/repo-config-docs/latest/updating.md).

3. Install new version packages. See [Installing AXDB](installing.md).

4. Restart the `postgresql` service:

    === ":material-debian: On Debian / Ubuntu"

         ```{.bash data-prompt="$"}
         $ sudo systemctl start postgresql.service
         ```

    === ":material-redhat: On Red Hat Enterprise Linux / derivatives"

         ```{.bash data-prompt="$"}
         $ sudo systemctl start postgresql-18
         ```

5. If you use `pg_tde`, update the extension. After restarting the cluster, connect to each database where `pg_tde` is installed and run:

    ```sql
    ALTER EXTENSION pg_tde UPDATE;
    ```

    This updates the extension to the latest installed version and needs to be run in each database where the extension is installed.

!!! note "For minor upgrades (RHEL only)"

    During a minor upgrade on RHEL, you may encounter the following error:

    ```
    Unknown Error occurred: Transaction test error:
    file /usr/share/postgresql-common/server/postgresql.mk from install of axdb-postgresql-common conflicts with file from package axdb-postgresql-common-dev
    file /usr/share/postgresql-common/t/040_upgrade.t from install of axdb-postgresql-common conflicts with file from package axdb-postgresql-common-dev
    ```

    To resolve this, remove the `axdb-postgresql-common-dev` package and reinstall it with the new intended upgraded server.
