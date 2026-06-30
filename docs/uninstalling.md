# Uninstalling AXDB

To uninstall AXDB, remove all the installed packages and data / configuration files.

!!! note
     Should you need the data files later, back up your data before uninstalling AXDB.

## Uninstall from tarballs

Stop the PostgreSQL server and remove the folder with the binary tarballs.

1. Stop the `postgres` server:

    ```{.bash data-prompt="$"}
    $ /path/to/tarballs/axdb{{pgversion}}/bin/pg_ctl -D path/to/datadir -l logfile stop
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        waiting for server to shut down.... done
        server stopped
        ```

2. Remove the directory with extracted tarballs

    ```{.bash data-prompt="$"}
    $ sudo rm -rf /path/to/tarballs/
    ```
