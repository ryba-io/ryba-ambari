
# Ambari Agent Configuration

    module.exports = (service) ->
      options = service.options


## SSL

      options.ssl = merge {}, service.deps.ssl?.options, options.ssl
      options.ssl.enabled ?= !!service.deps.ssl
      options.ssl.conf_dir ?= '/etc/security/serverKeys'
      options.iptables = !!service.deps.iptables and ((service.deps.iptables?.options.action is 'start') or (service.deps.iptables?.options.state is 'started'))
      if options.ssl.enabled
        options.ssl_client ?= {}
        options.ssl_server ?= {}
        throw Error "Required Option: ssl.cacert" if not options.ssl.cacert
        throw Error "Required Option: ssl.key" if not options.ssl.key
        throw Error "Required Option: ssl.cert" if  not options.ssl.cert
      options.java_home ?= service.deps.java.options.java_home
      options.jre_home ?= service.deps.java.options.java_home


## Environment

      options.fqdn = service.node.fqdn
      options.hostname = service.node.hostname
      options.only ?= []

## Services

      options.services ?= {}
      options.configurations ?= {}
      options.users ?= {}
      options.groups ?= {}

## ZOOKEEPER Service

      if service.deps.zookeeper_server?.length > 0
        for config_type in ['zoo.cfg', 'zookeeper-env']
          options.configurations["#{config_type}"] ?= {}
        options.zookeeper_user = merge {}, service.deps.zookeeper_server[0].options.user
        options.zookeeper_group = merge {}, service.deps.zookeeper_server[0].options.group
        options.services['ZOOKEEPER'] ?= {}
        options.services['ZOOKEEPER']['ZOOKEEPER_SERVER'] ?= {}
        options.services['ZOOKEEPER']['ZOOKEEPER_SERVER']['hosts'] ?= service.deps.zookeeper_server.map (srv) -> srv.node.fqdn
        exports.enrich_config service.deps.zookeeper_server[0].options.config, options.configurations['zoo.cfg']
        options.zookeeper_peer_port ?= service.deps.zookeeper_server[0].options.peer_port
        options.zookeeper_leader_port ?= service.deps.zookeeper_server[0].options.leader_port
        options.is_zookeeper_observer = service.deps.zookeeper_server.filter( (srv) -> srv.options.config.peerType is 'observer' ).map (srv) -> srv.node.fqdn
        options.zookeeper = options.services['ZOOKEEPER']['ZOOKEEPER_SERVER']['hosts'].indexOf(service.node.fqdn) > -1

## Ambari Infra Service

      if service.deps.ambari_infra_service?.length > 0
        options.configurations['infra-solr-env'] ?= {}
        options.services['AMBARI_INFRA'] ?= {}
        if service.deps.ambari_infra_instance?.length > 0
          options.services['AMBARI_INFRA']['INFRA_SOLR'] ?= {}
          options.services['AMBARI_INFRA']['INFRA_SOLR']['hosts'] ?= service.deps.ambari_infra_service.map (srv) -> srv.node.fqdn
          options.ambari_infra = options.services['AMBARI_INFRA']['INFRA_SOLR']['hosts'].indexOf(service.node.fqdn) > -1
          exports.enrich_config service.deps.ambari_infra_service[0].options.configurations['infra-solr-env'], options.configurations['infra-solr-env']
          exports.enrich_config service.deps.ambari_infra_instance[0].options.configurations['infra-solr-env'], options.configurations['infra-solr-env']
          options.users['ambari-infra'] ?= service.deps.ambari_infra_service[0].options.user
          options.groups['ambari-infra'] ?= service.deps.ambari_infra_service[0].options.group

## HDFS Service

      if service.deps.hdfs?.length > 0
        options.services['HDFS'] ?= {}
        for config_type in ['core-site','hdfs-site','hadoop-env', 'ssl-server', 'ssl-client', 'hadoop-policy']
          options.configurations["#{config_type}"] ?= {}
        if service.deps.hadoop_core?.length > 0
          options.hadoop_group = merge {}, service.deps.hdfs[0].options.hadoop_group, options.hadoop_group
          options.hdfs_user = merge {}, service.deps.hdfs[0].options.hdfs.user
          options.hdfs_group = merge {}, service.deps.hdfs[0].options.hdfs.group
          exports.enrich_config service.deps.hadoop_core[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hadoop_core[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hadoop_core[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          exports.enrich_config service.deps.hadoop_core[0].options.ssl_server, options.configurations['ssl-server']
          exports.enrich_config service.deps.hadoop_core[0].options.ssl_client, options.configurations['ssl-client']
          exports.enrich_config service.deps.hadoop_core[0].options.hadoop_policy, options.configurations['hadoop-policy']

        if service.deps.hdfs_client?.length > 0
          options.services['HDFS']['HDFS_CLIENT'] ?= {} 
          options.services['HDFS']['HDFS_CLIENT']['hosts'] = service.deps.hdfs_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_client[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_client[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_client[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
        if service.deps.hdfs_nn?.length > 0
          options.nameservice  = service.deps.hdfs_nn[0].options.nameservice
          options.services['HDFS']['NAMENODE'] ?= {}
          options.services['HDFS']['NAMENODE']['hosts'] = service.deps.hdfs_nn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_nn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_nn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_nn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          if options.services['HDFS']['NAMENODE']['hosts'].indexOf(service.node.fqdn) > -1
            options.hdfs_nn_ssl_server ?= service.deps.hdfs_nn[0].options.ssl_server
            options.hdfs_nn_ssl_client ?= service.deps.hdfs_nn[0].options.ssl_client
            options.hdfs_nn = options.services['HDFS']['NAMENODE']['hosts'].indexOf(service.node.fqdn) > -1
        if service.deps.hdfs_dn?.length > 0
          options.services['HDFS']['DATANODE'] ?= {}
          options.services['HDFS']['DATANODE']['hosts'] = service.deps.hdfs_dn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_dn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_dn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_dn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          options.hdfs_dn = options.services['HDFS']['DATANODE']['hosts'].indexOf(service.node.fqdn) > -1
        if service.deps.hdfs_jn?.length > 0
          options.services['HDFS']['JOURNALNODE'] ?= {}
          options.services['HDFS']['JOURNALNODE']['hosts'] = service.deps.hdfs_jn.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_jn[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_jn[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_jn[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          options.hdfs_jn = options.services['HDFS']['JOURNALNODE']['hosts'].indexOf(service.node.fqdn) > -1
        if service.deps.hdfs_zkfc?.length > 0
          options.services['HDFS']['ZKFC'] ?= {}
          options.services['HDFS']['ZKFC']['hosts'] = service.deps.hdfs_zkfc.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hdfs_zkfc[0].options.core_site, options.configurations['core-site']
          exports.enrich_config service.deps.hdfs_zkfc[0].options.hdfs_site, options.configurations['hdfs-site']
          exports.enrich_config service.deps.hdfs_zkfc[0].options.configurations['hadoop-env'], options.configurations['hadoop-env']
          options.hdfs_zkfc = options.services['HDFS']['ZKFC']['hosts'].indexOf(service.node.fqdn) > -1

## YARN Service

      if service.deps.yarn?.length > 0
        options.services['YARN'] ?= {}
        for config_type in ['yarn-site', 'yarn-env', 'capacity-scheduler']
          options.configurations["#{config_type}"] ?= {}
        options.yarn_user = merge {}, service.deps.yarn[0].options.yarn.user
        options.yarn_group = merge {}, service.deps.yarn[0].options.yarn.group
        exports.enrich_config service.deps.yarn[0].options.configurations['yarn-site'], options.configurations['yarn-site']
        exports.enrich_config service.deps.yarn[0].options.configurations['yarn-env'], options.configurations['yarn-env']
        if service.deps.yarn_client?.length > 0
          options.services['YARN']['YARN_CLIENT'] ?= {} 
          options.services['YARN']['YARN_CLIENT']['hosts'] = service.deps.yarn_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_client[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_client[0].options.configurations['yarn-env'], options.configurations['yarn-env']
        if service.deps.yarn_rm?.length > 0
          options.services['YARN']['RESOURCEMANAGER'] ?= {} 
          options.services['YARN']['RESOURCEMANAGER']['hosts'] = service.deps.yarn_rm.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_rm[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_rm[0].options.configurations['yarn-env'], options.configurations['yarn-env']
          exports.enrich_config service.deps.yarn_rm[0].options.capacity_scheduler, options.configurations['capacity-scheduler']
          options.yarn_rm = options.services['YARN']['RESOURCEMANAGER']['hosts'].indexOf(service.node.fqdn) > -1
        if service.deps.yarn_nm?.length > 0
          options.services['YARN']['NODEMANAGER'] ?= {} 
          options.services['YARN']['NODEMANAGER']['hosts'] = service.deps.yarn_nm.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_nm[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_nm[0].options.configurations['yarn-env'], options.configurations['yarn-env']
          options.yarn_nm = options.services['YARN']['NODEMANAGER']['hosts'].indexOf(service.node.fqdn) > -1
          options.cgroup = service.deps.yarn_nm[0].options.cgroup
        if service.deps.yarn_ts?.length > 0
          options.services['YARN']['APP_TIMELINE_SERVER'] ?= {} 
          options.services['YARN']['APP_TIMELINE_SERVER']['hosts'] = service.deps.yarn_ts.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.yarn_ts[0].options.yarn_site, options.configurations['yarn-site']
          exports.enrich_config service.deps.yarn_ts[0].options.configurations['yarn-env'], options.configurations['yarn-env']
          options.yarn_ts = options.services['YARN']['APP_TIMELINE_SERVER']['hosts'].indexOf(service.node.fqdn) > -1

## MAPREDUCE2 Service

      if service.deps.mapreduce?.length > 0
        options.services['MAPREDUCE2'] ?= {}
        for config_type in ['mapred-site', 'mapred-env']
          options.configurations["#{config_type}"] ?= {}
        options.mapred_user = merge {}, service.deps.mapreduce[0].options.mapred.user
        options.mapred_group = merge {}, service.deps.mapreduce[0].options.mapred.group
        if service.deps.mapred_client?.length > 0
          options.services['MAPREDUCE2']['MAPREDUCE2_CLIENT'] ?= {} 
          options.services['MAPREDUCE2']['MAPREDUCE2_CLIENT']['hosts'] = service.deps.mapred_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.mapred_client[0].options.mapred_site, options.configurations['mapred-site']
          exports.enrich_config service.deps.mapred_client[0].options.configurations['mapred-env'], options.configurations['mapred-env']
          options.mapreduce = options.services['MAPREDUCE2']['MAPREDUCE2_CLIENT']['hosts'].indexOf(service.node.fqdn) > -1
        if service.deps.mapred_jhs?.length > 0
          options.services['MAPREDUCE2']['HISTORY_SERVER'] ?= {} 
          options.services['MAPREDUCE2']['HISTORY_SERVER']['hosts'] = service.deps.mapred_jhs.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.mapred_jhs[0].options.mapred_site, options.configurations['mapred-site']
          exports.enrich_config service.deps.mapred_jhs[0].options.configurations['mapred-env'], options.configurations['mapred-env']
          options.mapred_jhs = options.services['MAPREDUCE2']['HISTORY_SERVER']['hosts'].indexOf(service.node.fqdn) > -1

## HBASE Service

      if service.deps.hbase_service?.length > 0
        options.services['HBASE'] ?= {}
        for config_type in ['hbase-site', 'hbase-env', 'hbase-policy']
          options.configurations["#{config_type}"] ?= {}
        options.hbase_user = merge {}, service.deps.hbase_service[0].options.user
        options.hbase_group = merge {}, service.deps.hbase_service[0].options.group
        exports.enrich_config service.deps.hbase_service[0].options.configurations['hbase-site'], options.configurations['hbase-site']
        exports.enrich_config service.deps.hbase_service[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_client?.length > 0
          options.services['HBASE']['HBASE_CLIENT'] ?= {} 
          options.services['HBASE']['HBASE_CLIENT']['hosts'] = service.deps.hbase_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_client[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_client[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_master?.length > 0
          options.services['HBASE']['HBASE_MASTER'] ?= {} 
          options.services['HBASE']['HBASE_MASTER']['hosts'] = service.deps.hbase_master.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_master[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_master[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.hbase_regionserver?.length > 0
          options.hbase_regionserver = service.deps.hbase_regionserver.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['HBASE']['HBASE_REGIONSERVER'] ?= {} 
          options.services['HBASE']['HBASE_REGIONSERVER']['hosts'] = service.deps.hbase_regionserver.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hbase_regionserver[0].options.hbase_site, options.configurations['hbase-site']
          exports.enrich_config service.deps.hbase_regionserver[0].options.configurations['hbase-env'], options.configurations['hbase-env']
        if service.deps.phoenix_queryserver?.length > 0
          options.services['HBASE']['PHOENIX_QUERY_SERVER'] ?= {} 
          options.services['HBASE']['PHOENIX_QUERY_SERVER']['hosts'] ?= service.deps.phoenix_queryserver.map (srv) -> srv.node.fqdn 
          exports.enrich_config service.deps.phoenix_queryserver[0].options.configurations['hbase-site'], options.configurations['hbase-site']
          options.phoenix_queryserver = service.deps.phoenix_queryserver.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
        if service.deps.phoenix_client?.length > 0
          exports.enrich_config service.deps.phoenix_client[0].options.configurations['hbase-site'], options.configurations['hbase-site']
        if service.deps.hbase_rest?.length > 0
          krb5_username = /^(.+?)[@\/]/.exec(service.deps.hbase_rest[0].options.hbase_site['hbase.rest.kerberos.principal'])?[1]
          throw Error 'Invalid HBase Rest principal' unless krb5_username
          options.configurations['hbase-site']["hadoop.proxyuser.#{krb5_username}.groups"] ?= '*'
          options.configurations['hbase-site']["hadoop.proxyuser.#{krb5_username}.hosts"] ?= '*'
          options.configurations['core-site']["hadoop.proxyuser.#{krb5_username}.groups"] ?= '*'
          options.configurations['core-site']["hadoop.proxyuser.#{krb5_username}.hosts"] ?= '*'

## HIVE Service

      if service.deps.hive?.length > 0
        options.services['HIVE'] ?= {}
        for config_type in ['hive-site', 'hive-env', 'webhcat-site']
          options.configurations["#{config_type}"] ?= {}
        exports.enrich_config service.deps.hive[0].options.configurations['hive-site'], options.configurations['hive-site']
        exports.enrich_config service.deps.hive[0].options.configurations['hive-env'], options.configurations['hive-env']
        options.hive_user = merge {}, service.deps.hive[0].options.user
        options.hive_group = merge {}, service.deps.hive[0].options.group
        if service.deps.hive_client?.length > 0
          options.services['HIVE']['HIVE_CLIENT'] ?= {} 
          options.services['HIVE']['HIVE_CLIENT']['hosts'] = service.deps.hive_client.map (srv) -> srv.node.fqdn
          options.services['HIVE']['HCAT'] ?= {}
          options.services['HIVE']['HCAT']['hosts'] ?= []
          options.services['HIVE']['HCAT']['hosts'] = service.deps.hive_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hive_client[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_client[0].options.configurations['hive-env'], options.configurations['hive-env']
          options.hive_client = service.deps.hive_client.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.hive_client_truststore_location ?= service.deps.hive_client[0].options.truststore_location
          options.hive_client_truststore_password ?= service.deps.hive_client[0].options.truststore_password
        if service.deps.hive_beeline?.length > 0
          options.services['HIVE']['HIVE_CLIENT'] ?= {} 
          options.services['HIVE']['HIVE_CLIENT']['hosts'] ?= []
          options.services['HIVE']['HIVE_CLIENT']['hosts'].push service.deps.hive_beeline.map( (srv) -> srv.node.fqdn)...
          options.services['HIVE']['HCAT'] ?= {}
          options.services['HIVE']['HCAT']['hosts'] ?= []
          options.services['HIVE']['HCAT']['hosts'].push service.deps.hive_beeline.map( (srv) -> srv.node.fqdn)...
          exports.enrich_config service.deps.hive_beeline[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_beeline[0].options.configurations['hive-env'], options.configurations['hive-env']
          options.hive_client =  (options.hive_client || service.deps.hive_beeline.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1)
          options.hive_client_truststore_location ?= service.deps.hive_beeline[0].options.truststore_location
          options.hive_client_truststore_password ?= service.deps.hive_beeline[0].options.truststore_password
        if service.deps.hive_server2?.length > 0
          options.services['HIVE']['HIVE_SERVER'] ?= {} 
          options.services['HIVE']['HIVE_SERVER']['hosts'] = service.deps.hive_server2.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hive_server2[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hive_server2[0].options.configurations['hive-env'], options.configurations['hive-env']
          options.hive_server2 = service.deps.hive_server2.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
        if service.deps.hcatalog?.length > 0
          options.hive_hcatalog = service.deps.hcatalog.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['HIVE']['HIVE_METASTORE'] ?= {} 
          options.services['HIVE']['HIVE_METASTORE']['hosts'] = service.deps.hcatalog.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.hcatalog[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.hcatalog[0].options.configurations['hive-env'], options.configurations['hive-env']
          options.hcat_user ?= service.deps.hcatalog[0].options.user
          options.hcat_group ?= service.deps.hcatalog[0].options.group
          options.hive_db ?= service.deps.hive_metastore[0].options.db
        if service.deps.webhcat?.length > 0
          options.webhcat_user = merge {}, service.deps.webhcat[0].options.user
          options.webhcat_group = merge {}, service.deps.webhcat[0].options.group
          options.hive_webhcat = service.deps.webhcat.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['HIVE']['WEBHCAT_SERVER'] ?= {}
          options.services['HIVE']['WEBHCAT_SERVER']['hosts'] ?= service.deps.webhcat.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.webhcat[0].options.hive_site, options.configurations['hive-site']
          exports.enrich_config service.deps.webhcat[0].options.configurations['hive-env'], options.configurations['hive-env']
          exports.enrich_config service.deps.webhcat[0].options.webhcat_site, options.configurations['webhcat-site']

## Oozie Service

      if service.deps.oozie_service?.length > 0
        options.services['OOZIE'] ?= {}
        for config_type in ['oozie-site', 'oozie-env']
          options.configurations["#{config_type}"] ?= {}
        exports.enrich_config service.deps.oozie_service[0].options.configurations['oozie-site'], options.configurations['oozie-site']
        exports.enrich_config service.deps.oozie_service[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        options.oozie_user ?= merge {}, service.deps.oozie_service[0].options.user
        options.oozie_group ?= merge {}, service.deps.oozie_service[0].options.group
        options.oozie_ssl ?= service.deps.oozie_service[0].options.ssl
        if service.deps.oozie_server.length > 0
          options.services['OOZIE']['OOZIE_SERVER'] ?= {} 
          options.services['OOZIE']['OOZIE_SERVER']['hosts'] = service.deps.oozie_server.map (srv) -> srv.node.fqdn
          options.oozie_server = options.services['OOZIE']['OOZIE_SERVER']['hosts'].indexOf(service.node.fqdn) > -1
          exports.enrich_config service.deps.oozie_server[0].options.oozie_site, options.configurations['oozie-site']
          exports.enrich_config service.deps.oozie_server[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        if service.deps.oozie_client.length > 0
          service.deps.oozie_client[0].options.configurations ?= {}
          options.services['OOZIE']['OOZIE_CLIENT'] ?= {} 
          options.services['OOZIE']['OOZIE_CLIENT']['hosts'] = service.deps.oozie_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.oozie_client[0].options.oozie_site, options.configurations['oozie-site']
          exports.enrich_config service.deps.oozie_client[0].options.configurations['oozie-env'], options.configurations['oozie-env']
        options.configurations['core-site']["hadoop.proxyuser.#{options.oozie_user.name}.groups"] ?= '*'
        options.configurations['core-site']["hadoop.proxyuser.#{options.oozie_user.name}.hosts"] ?= service.deps.oozie_server.map (srv) -> srv.node.fqdn 

## RANGER Service

      if service.deps.ranger_hdpadmin?.length > 0
        options.services['RANGER'] ?= {}
        options.users['ranger'] ?= merge {}, service.deps.ranger_hdpadmin[0].options.user
        options.groups['ranger'] ?= merge {}, service.deps.ranger_hdpadmin[0].options.group
        options.ranger_admin = service.deps.ranger_hdpadmin.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
        options.configurations['ranger-admin-site'] ?= merge {}, service.deps.ranger_hdpadmin[0].options.configurations['ranger-admin-site']
        
### Ranger Plugins

        if service.deps.ranger_hdfs?.length > 0
          if service.deps.ranger_hdfs.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-hdfs-policymgr-ssl'] ?= merge {}, service.deps.ranger_hdfs[0].options.configurations['ranger-hdfs-policymgr-ssl'], options.configurations['ranger-hdfs-policymgr-ssl']
        if service.deps.ranger_yarn?.length > 0
          if service.deps.ranger_yarn.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-yarn-policymgr-ssl'] ?= merge {}, service.deps.ranger_yarn[0].options.configurations['ranger-yarn-policymgr-ssl'], options.configurations['ranger-yarn-policymgr-ssl']
        if service.deps.ranger_hive?.length > 0
          if service.deps.ranger_hive.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-hive-policymgr-ssl'] ?= merge {}, service.deps.ranger_hive[0].options.configurations['ranger-hive-policymgr-ssl'], options.configurations['ranger-hive-policymgr-ssl']
        if service.deps.ranger_hbase?.length > 0
          if service.deps.ranger_hbase.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-hbase-policymgr-ssl'] ?= merge {}, service.deps.ranger_hbase[0].options.configurations['ranger-hbase-policymgr-ssl'], options.configurations['ranger-hbase-policymgr-ssl']
        if service.deps.ranger_kafka?.length > 0
          if service.deps.ranger_kafka.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-kafka-policymgr-ssl'] ?= merge {}, service.deps.ranger_kafka[0].options.configurations['ranger-kafka-policymgr-ssl'], options.configurations['ranger-kafka-policymgr-ssl']
        if service.deps.ranger_knox?.length > 0
          if service.deps.ranger_knox.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-knox-policymgr-ssl'] ?= merge {}, service.deps.ranger_knox[0].options.configurations['ranger-knox-policymgr-ssl'], options.configurations['ranger-knox-policymgr-ssl']
        if service.deps.ranger_atlas?.length > 0
          if service.deps.ranger_atlas.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
            options.configurations['ranger-atlas-policymgr-ssl'] ?= merge {}, service.deps.ranger_atlas[0].options.configurations['ranger-atlas-policymgr-ssl'], options.configurations['ranger-atlas-policymgr-ssl']

## Ambari Metrics Service

      if service.deps.ambari_metrics_service?.length > 0
        options.services['AMBARI_METRICS'] ?= {}
        options.configurations['ams-env'] ?= {}
        options.configurations['ams-grafana-ini'] ?= {}
        options.configurations['ams-grafana-env'] ?= {}
        options.configurations['ams-hbase-security-site'] ?= {}
        options.ams_user = service.deps.ambari_metrics_service[0].options.user
        options.ams_group = service.deps.ambari_metrics_service[0].options.group
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-env'], options.configurations['ams-env']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
        exports.enrich_config service.deps.ambari_metrics_service[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_collector.length > 0
          options.services['AMBARI_METRICS']['METRICS_COLLECTOR'] ?= {} 
          options.services['AMBARI_METRICS']['METRICS_COLLECTOR']['hosts'] = service.deps.ambari_metrics_collector.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_collector[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_monitor.length > 0
          options.services['AMBARI_METRICS']['METRICS_MONITOR'] ?= {} 
          options.services['AMBARI_METRICS']['METRICS_MONITOR']['hosts'] = service.deps.ambari_metrics_monitor.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_monitor[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        if service.deps.ambari_metrics_grafana.length > 0
          options.services['AMBARI_METRICS']['METRICS_GRAFANA'] ?= {}
          options.services['AMBARI_METRICS']['METRICS_GRAFANA']['hosts'] = service.deps.ambari_metrics_grafana.map (srv) -> srv.node.fqdn
          options.ambari_grafana = options.services['AMBARI_METRICS']['METRICS_GRAFANA']['hosts'].indexOf(service.node.fqdn) > -1
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-env'], options.configurations['ams-env']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-grafana-ini'], options.configurations['ams-grafana-ini']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-grafana-env'], options.configurations['ams-grafana-env']
          exports.enrich_config service.deps.ambari_metrics_grafana[0].options.configurations['ams-hbase-security-site'], options.configurations['ams-hbase-security-site']
        # options.configurations['core-site']["hadoop.proxyuser.#{options.ams_user.name}.groups"] ?= '*'
        # options.configurations['core-site']["hadoop.proxyuser.#{options.ams_user.name}.hosts"] ?= service.deps.ambari_metrics_monitor.map (srv) -> srv.node.fqdn 

## LOGSEARCH Service

      if service.deps.logsearch_service?.length > 0
        options.services['LOGSEARCH'] ?= {}
        options.configurations['logfeeder-env'] ?= {}
        options.configurations['logsearch-env'] ?= {}
        options.configurations['logsearch-env'] ?= {}
        options.configurations['logsearch-common-env'] ?= {}
        options.logsearch_user = service.deps.logsearch_service[0].options.user
        options.logsearch_group = service.deps.logsearch_service[0].options.group
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        if service.deps.logsearch_server.length > 0
          options.services['LOGSEARCH']['LOGSEARCH_SERVER'] ?= {} 
          options.services['LOGSEARCH']['LOGSEARCH_SERVER']['hosts'] = service.deps.logsearch_server.map (srv) -> srv.node.fqdn
          options.logsearch_server = options.services['LOGSEARCH']['LOGSEARCH_SERVER']['hosts'].indexOf(service.node.fqdn) > -1
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
          exports.enrich_config service.deps.logsearch_server[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        exports.enrich_config service.deps.logsearch_service[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']
        if service.deps.logsearch_feeder.length > 0
          options.services['LOGSEARCH']['LOGSEARCH_LOGFEEDER'] ?= {} 
          options.services['LOGSEARCH']['LOGSEARCH_LOGFEEDER']['hosts'] = service.deps.logsearch_feeder.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logfeeder-env'], options.configurations['logfeeder-env']
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logsearch-common-env'], options.configurations['logsearch-common-env']
          exports.enrich_config service.deps.logsearch_feeder[0].options.configurations['logsearch-env'], options.configurations['logsearch-env']

## SMARTSENSE Service

      if service.deps.smartsense_service?.length > 0
        options.services['SMARTSENSE'] ?= {}
        options.configurations['activity-zeppelin-site'] ?= {}
        options.smartsense_user = service.deps.smartsense_service[0].options.user
        options.smartsense_group = service.deps.smartsense_service[0].options.group
        if service.deps.smartsense_explorer.length > 0
          options.smartsense_explorer = service.deps.smartsense_explorer.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['SMARTSENSE']['ACTIVITY_EXPLORER'] ?= {} 
          options.services['SMARTSENSE']['ACTIVITY_EXPLORER']['hosts'] = service.deps.smartsense_explorer.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_explorer[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']
        if service.deps.smartsense_server.length > 0
          options.services['SMARTSENSE']['HST_SERVER'] ?= {} 
          options.services['SMARTSENSE']['HST_SERVER']['hosts'] = service.deps.smartsense_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_server[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']
        if service.deps.smartsense_agent.length > 0
          options.services['SMARTSENSE']['HST_AGENT'] ?= {} 
          options.services['SMARTSENSE']['HST_AGENT']['hosts'] = service.deps.smartsense_agent.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.smartsense_agent[0].options.configurations['activity-zeppelin-site'], options.configurations['activity-zeppelin-site']

## SPARK 1 Service

      if service.deps.spark_service?.length > 0
        options.services['SPARK'] ?= {}
        options.configurations['spark-defaults'] ?= {}
        options.configurations['spark-env'] ?= {}
        options.spark_user = service.deps.spark_service[0].options.user
        options.spark_group = service.deps.spark_service[0].options.group
        if service.deps.spark_hs?.length > 0
          options.services['SPARK']['SPARK_JOBHISTORYSERVER'] ?= {} 
          options.services['SPARK']['SPARK_JOBHISTORYSERVER']['hosts'] = service.deps.spark_hs.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark_hs[0].options.conf, options.configurations['spark-defaults']
        if service.deps.spark_client?.length > 0
          options.services['SPARK']['SPARK_CLIENT'] ?= {} 
          options.services['SPARK']['SPARK_CLIENT']['hosts'] = service.deps.spark_client.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.spark_client[0].options.conf, options.configurations['spark-defaults']

## SQOOP Service

      if service.deps.sqoop?.length > 0
        options.sqoop_user = service.deps.sqoop[0].options.user
        options.sqoop_group = service.deps.sqoop[0].options.group
        options.configurations['sqoop-site'] ?= {}
        options.services['SQOOP'] ?= {}
        options.services['SQOOP']['SQOOP'] ?= {}
        options.services['SQOOP']['SQOOP']['hosts'] = service.deps.sqoop.map (srv) -> srv.node.fqdn

## TEZ Service

      if service.deps.tez?.length > 0
        options.configurations['tez-site'] ?= {}
        options.tez = true
        options.tez_user = service.deps.tez[0].options.user
        options.tez_group = service.deps.tez[0].options.group
        options.services['TEZ'] ?= {}
        options.services['TEZ']['TEZ_CLIENT'] ?= {}
        options.services['TEZ']['TEZ_CLIENT']['hosts'] = service.deps.tez.map (srv) -> srv.node.fqdn
        exports.enrich_config service.deps.tez[0].options.configurations['tez-site'] , options.configurations['tez-site']
        
## KNOX Service

      if service.deps.knox_service?.length > 0
        options.services['KNOX'] ?= {}
        options.configurations['gateway-site'] ?= {}
        options.knox_user = service.deps.knox_service[0].options.user
        options.knox_group = service.deps.knox_service[0].options.group
        if service.deps.knox_server?.length > 0
          options.knox_importCerts ?= service.deps.knox_service[0].options.importCerts
          options.knox_server = service.deps.knox_server.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['KNOX']['KNOX_GATEWAY'] ?= {} 
          options.services['KNOX']['KNOX_GATEWAY']['hosts'] = service.deps.knox_server.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.knox_server[0].options.configurations['gateway-site'], options.configurations['gateway-site']

## KAFKA Service

      if service.deps.kafka_service?.length > 0
        options.services['KAFKA'] ?= {}
        options.configurations['kafka-broker'] ?= {}
        options.configurations['kafka-env'] ?= {}
        options.configurations['kafka-log4j'] ?= {}
        options.kafka_user = service.deps.kafka_service[0].options.user
        options.kafka_group = service.deps.kafka_service[0].options.group
        exports.enrich_config service.deps.kafka_service[0].options.config, options.configurations['kafka-broker']
        exports.enrich_config service.deps.kafka_service[0].options.configurations['kafka-env'], options.configurations['kafka-env']
        if service.deps.kafka_broker.length > 0
          options.services['KAFKA']['KAFKA_BROKER'] ?= {} 
          options.services['KAFKA']['KAFKA_BROKER']['hosts'] = service.deps.kafka_broker.map (srv) -> srv.node.fqdn
          options.kafka_broker = service.deps.kafka_broker.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.kafka_protocols = service.deps.kafka_broker[0].options.protocols
          options.kafka_ports = service.deps.kafka_broker[0].options.ports
          exports.enrich_config service.deps.kafka_broker[0].options.config, options.configurations['kafka-broker']
          exports.enrich_config service.deps.kafka_broker[0].options.configurations['kafka-env'], options.configurations['kafka-env']
          options.kafka_env ?= service.deps.kafka_broker[0].options.env

## Zeppelin

      if service.deps.zeppelin_service?.length > 0
        options.services['ZEPPELIN'] ?= {}
        options.configurations['zeppelin-config'] ?= {}
        options.configurations['zeppelin-env'] ?= {}
        options.zeppelin_user = service.deps.zeppelin_service[0].options.user
        options.zeppelin_group = service.deps.zeppelin_service[0].options.group
        if service.deps.zeppelin_master?.length > 0
          options.zeppelin_master = service.deps.zeppelin_master.map( (srv) -> srv.node.fqdn ).indexOf(service.node.fqdn) > -1
          options.services['ZEPPELIN']['ZEPPELIN_MASTER'] ?= {} 
          options.services['ZEPPELIN']['ZEPPELIN_MASTER']['hosts'] = service.deps.zeppelin_master.map (srv) -> srv.node.fqdn
          exports.enrich_config service.deps.zeppelin_master[0].options.configurations['zeppelin-config'], options.configurations['zeppelin-config']
          exports.enrich_config service.deps.zeppelin_master[0].options.configurations['zeppelin-env'], options.configurations['zeppelin-env']

### User Provisionning
Contains object of user that ambari-agent should create on all hosts. By default
Ambari needs to all user on all node even if the service is not installed on a host.

The components should register their user to ambari agents

      options.users['hdfs'] ?= options.hdfs_user
      options.users['yarn'] ?= options.yarn_user
      options.users['mapred'] ?= options.mapred_user
      options.users['zookeeper'] ?= options.zookeeper_user
      options.users['hbase'] ?= options.hbase_user
      options.users['hive'] ?= options.hive_user
      options.users['ams'] ?= options.ams_user
      options.users['oozie'] ?= options.oozie_user
      options.users['logsearch'] ?= options.logsearch_user
      options.users['ranger'] ?= options.ranger_user
      options.users['smartsense'] ?= options.smartsense_user
      options.users['spark'] ?= options.spark_user
      options.users['tez'] ?= options.tez_user
      options.users['sqoop'] ?= options.sqoop_user
      options.users['kafka'] ?= options.kafka_user
      options.users['zeppelin'] ?= options.zeppelin_user
      options.users['hcat'] ?= options.hcat_user
      options.users['webhcat'] ?= options.webhcat_user
      options.groups['hadoop_group'] ?= options.hadoop_group
      options.groups['hdfs'] ?= options.hdfs_group
      options.groups['yarn'] ?= options.yarn_group
      options.groups['mapred'] ?= options.mapred_group
      options.groups['zookeeper'] ?= options.zookeeper_group
      options.groups['hbase'] ?= options.hbase_group
      options.groups['hive'] ?= options.hive_group
      options.groups['ams'] ?= options.ams_group
      options.groups['logsearch'] ?= options.logsearch_group
      options.groups['oozie'] ?= options.oozie_group
      options.groups['ranger'] ?= options.ranger_group
      options.groups['smartsense'] ?= options.smartsense_group
      options.groups['spark'] ?= options.spark_group
      options.groups['tez'] ?= options.tez_group
      options.groups['sqoop'] ?= options.sqoop_group
      options.groups['kafka'] ?= options.kafka_group
      options.groups['zeppelin'] ?= options.zeppelin_group
      options.groups['hcat'] ?= options.hcat_group
      options.groups['webhcat'] ?= options.webhcat_group

## Utilities

    exports.enrich_config = (source, target) ->
      target ?= {}
      for k, v of source
        target[k] ?= v

## Dependencies

    {merge} = require 'nikita/lib/misc'
