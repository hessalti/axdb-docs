# Migrate from PostgreSQL to AXDB 


AXDB includes the PostgreSQL database and additional extensions that have been selected to cover the needs of the enterprise and are guaranteed to work together. AXDB is available as a software collection that is easy to deploy.

We encourage users to migrate from their PostgreSQL deployments based on community binaries to AXDB. This document provides the migration instructions. 

Depending on your business requirements, you may migrate to AXDB either [on the same server](#migrate-on-the-same-server) or [onto a different server](#migrate-on-a-different-server). 

## Migrate on the same server

=== ":material-redhat: On RHEL and derivatives"

       > To ensure that your data is safe during the migration, we recommend to make a backup of your data and all configuration files (such as `pg_hba.conf`, `postgresql.conf`, `postgresql.auto.conf`) using the tool of your choice. The backup process is out of scope of this document. You can use `pg_dumpall` or other tools of your choice. 

     Run **all** commands as root or via **sudo**:
    {.power-number}
    
    1. Stop the `postgresql` server   

          ```{.bash data-prompt="$"}
          $ sudo systemctl stop postgresql-{{pgversion}}
          ```

      2. Remove community packages

         ```{.bash data-prompt="$"}
         $ sudo yum remove postgresql
         ```

      3. [Install AXDB packages](tarball.md)
      4. (Optional) Restore the data from the backup.


## Migrate on a different server

In this scenario, we will refer to the server with PostgreSQL Community as the "source" and to the server with AXDB as the "target".

To migrate from PostgreSQL Community to AXDB on a different server, do the following:

**On the source server**:
{.power-number}

1. Back up your data and all configuration files (such as `pg_hba.conf`, `postgresql.conf`, `postgresql.auto.conf`) using the tool of your choice.
2. Stop the `postgresql` service

    === ":material-redhat: On RHEL and derivatives"

         ```{.bash data-prompt="$"}
         $ sudo systemctl stop postgresql-{{pgversion}}
         ```

3. Optionally, remove PostgreSQL Community packages 

**On the target server**:
{.power-number}

1. [Install AXDB packages](tarball.md) on the target server.
2. Restore the data from the backup
3. Start `postgresql`