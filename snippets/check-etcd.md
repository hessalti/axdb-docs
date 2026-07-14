3. Check the etcd cluster members. Use `etcdctl` for this purpose. Ensure that `etcdctl` interacts with etcd using API version 3 and knows which nodes, or endpoints, to communicate with. For this, we will define the required information as environment variables. Run the following commands on one of the nodes:

    ```
    export ETCDCTL_API=3
    HOST_1=192.168.3.201
    HOST_2=192.168.3.202
    HOST_3=192.168.3.203
    ENDPOINTS=$HOST_1:2379,$HOST_2:2379,$HOST_3:2379
    ```

4. Now, list the cluster members and output the result as a table as follows:
    
    ```{.bash data-prompt="$"}
    $ sudo /opt/axdb/axdb-etcd/bin/etcdctl --endpoints=$ENDPOINTS -w table member list
    ```

    ??? example "Sample output"

        ```
        +------------------+---------+-------+---------------------------+---------------------------+------------+
        |        ID        | STATUS  | NAME  |         PEER ADDRS        |        CLIENT ADDRS       | IS LEARNER |
        +------------------+---------+-------+---------------------------+---------------------------+------------+
        | 4788684035f976d3 | started | node2 | http://192.168.3.202:2380 | http://192.168.3.202:2379 |      false |
        | 67684e355c833ffa | started | node3 | http://192.168.3.203:2380 | http://192.168.3.203:2379 |      false |
        | 9d2e318af9306c67 | started | node1 | http://192.168.3.201:2380 | http://192.168.3.201:2379 |      false |
        +------------------+---------+-------+---------------------------+---------------------------+------------+
        ```

5. To check what node is currently the leader, use the following command

    ```{.bash data-prompt="$"}
    $ sudo /opt/axdb/axdb-etcd/bin/etcdctl --endpoints=$ENDPOINTS -w table endpoint status
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
        |      ENDPOINT      |        ID        | VERSION | DB SIZE | IS LEADER | IS LEARNER | RAFT TERM | RAFT INDEX | RAFT APPLIED INDEX | ERRORS |
        +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
        | 192.168.3.201:2379 | 9d2e318af9306c67 |  3.5.30 |   20 kB |      true |      false |         2 |         10 |                 10 |        |
        | 192.168.3.202:2379 | 4788684035f976d3 |  3.5.30 |   20 kB |     false |      false |         2 |         10 |                 10 |        |
        | 192.168.3.203:2379 | 67684e355c833ffa |  3.5.30 |   20 kB |     false |      false |         2 |         10 |                 10 |        |
        +--------------------+------------------+---------+---------+-----------+------------+-----------+------------+--------------------+--------+
        ```


6. To check the health of the etcd nodes, use the following command

    ```{.bash data-prompt="$"}
    $ sudo /opt/axdb/axdb-etcd/bin/etcdctl --endpoints=$ENDPOINTS -w table endpoint health
    ```

    ??? example "Sample output"

        ```{.text .no-copy}
        +--------------------+--------+------------+-------+
        |      ENDPOINT      | HEALTH |    TOOK    | ERROR |
        +--------------------+--------+------------+-------+
        | 192.168.3.201:2379 |   true | 2.208412ms |       |
        | 192.168.3.202:2379 |   true | 2.783703ms |       |
        | 192.168.3.203:2379 |   true | 48.64194ms |       |
        +--------------------+--------+------------+-------+
        ```

