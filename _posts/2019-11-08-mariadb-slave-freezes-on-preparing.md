---
layout: post
title:  "MariaDB replication issue - Slave freezes on preparing"
permalink: /mariadb-slave-freezes-on-preparing
date:   2019-08-11 13:00:00
tags: Databases Server Linux Mariadb
---

A few days ago a collegue reported this weird issue.
Two MariaDB servers, configured with a master-master replication, were updated to a newer available version. After the update (10.4.6 -> 10.4.7) the replication was not working anymore. Both replication servers were stuck at _preparing_.

{% highlight bash %}
MariaDB [(none)]> show slave status\G;
*************************** 1. row ***************************
                Slave_IO_State: NULL
                   Master_Host: data2
                   Master_User: replic
                   Master_Port: 3306
                 Connect_Retry: 60
               Master_Log_File: mariadb-bin.018865
           Read_Master_Log_Pos: 14137515
                Relay_Log_File: relay-bin.017274
                 Relay_Log_Pos: 4
         Relay_Master_Log_File: mariadb-bin.018865
              Slave_IO_Running: Preparing
             Slave_SQL_Running: Yes
               Replicate_Do_DB:
           Replicate_Ignore_DB:
            Replicate_Do_Table:
        Replicate_Ignore_Table:
       Replicate_Wild_Do_Table:
   Replicate_Wild_Ignore_Table: 
                    Last_Errno: 0
                    Last_Error:
                  Skip_Counter: 0
           Exec_Master_Log_Pos: 14137515
               Relay_Log_Space: 768
               Until_Condition: None
                Until_Log_File:
                 Until_Log_Pos: 0
            Master_SSL_Allowed: No
            Master_SSL_CA_File:
            Master_SSL_CA_Path:
               Master_SSL_Cert:
             Master_SSL_Cipher:
                Master_SSL_Key:
         Seconds_Behind_Master: NULL
 Master_SSL_Verify_Server_Cert: No
                 Last_IO_Errno: 0
                 Last_IO_Error:
                Last_SQL_Errno: 0
                Last_SQL_Error:
   Replicate_Ignore_Server_Ids:
              Master_Server_Id: 2
                Master_SSL_Crl:
            Master_SSL_Crlpath:
                    Using_Gtid: No
                   Gtid_IO_Pos:
       Replicate_Do_Domain_Ids:
   Replicate_Ignore_Domain_Ids:
                 Parallel_Mode: conservative
                     SQL_Delay: 0
           SQL_Remaining_Delay: NULL
       Slave_SQL_Running_State: Slave has read all relay log; waiting for the slave I/O thread to update it
              Slave_DDL_Groups: 0
Slave_Non_Transactional_Groups: 0
    Slave_Transactional_Groups: 0
1 row in set (0.000 sec)

ERROR: No query specified
{% endhighlight %}

This issue could not be solved by the usual fixed one would make after such a problem. After searching on various forums we found the same bug reported on the MariaDB Jira board 2 days ago. 

A possible hotfix would be changing the settings:
{% highlight bash %}
innodb_thread_concurrency = 0
{% endhighlight %}

Since this could lead to some other problems, a downgrade was needed, which i explained [here](/downgrade-mariadb).

