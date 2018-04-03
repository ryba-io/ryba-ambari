
# Hive HCatalog Start

Start the Hive HCatalog server. 

    module.exports =  header: 'Ambari Hive HCatalog Start', handler: (options) ->

## Registry

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'

## Wait

The Hive HCatalog require the database server to be started. The HDFS Namenode 
need to functionnal for Hive to answer queries.

      # console.log 'options.wait_db_admin', options.wait_db_admin
      @call 'masson/core/krb5_client/wait', once: true, options.wait_krb5_client
      @call 'ryba/zookeeper/server/wait', once: true, options.wait_zookeeper_server
      @call 'ryba-ambari-takeover/hadoop/hdfs_nn/wait', once: true, options.wait_hdfs_nn, conf_dir: options.hdfs_conf_dir
      @call 'ryba/commons/db_admin/wait', once: true, options.wait_db_admin

## Service

You can also start the server manually with the
following commands:

```
su hive -l -s /bin/bash -c 'cat /var/run/hive/hive.pid 1>/tmp/tmpPMCJrY 2>/tmp/tmpSPuQht'
su -l hive -c 'nohup hive --config /etc/hive-hcatalog/conf --service metastore >/var/log/hive-hcatalog/hcat.out 2>/var/log/hive-hcatalog/hcat.err & echo $! >/var/run/hive-hcatalog/hive-hcatalog.pid'
```

      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HIVE_METASTORE'
        hostname: options.fqdn

# Module Dependencies

    db = require 'nikita/lib/misc/db'
