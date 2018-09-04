# Ranger HBase Plugin

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hbase: module: 'ryba-ambari-takeover/hbase/service'
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'#, local: true
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver', local: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/hbase/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_MASTER'
        , ->
          @call 'ryba-ambari-takeover/ranger/plugins/hbase/install', options
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_REGIONSERVER'
        , ->
          @call 'ryba-ambari-takeover/ranger/plugins/hbase/install', options
      commands:
        install:
          'ryba-ambari-takeover/ranger/plugins/hbase/install'
