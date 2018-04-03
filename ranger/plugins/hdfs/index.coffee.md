
# Ranger HDFS Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/hdfs/configure'
      plugin: (options) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'NAMENODE'
        , ->
          delete options.original.type
          delete options.original.handler
          delete options.original.argument
          delete options.original.store
          @call 'ryba-ambari-takeover/ranger/plugins/hdfs/install', options.original
      commands:
        'install':
          'ryba-ambari-takeover/ranger/plugins/hdfs/install'
