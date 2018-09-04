# Ranger HDFS Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', local: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_rm_local: module: 'ryba-ambari-takeover/hadoop/yarn_rm', local: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs', required: true
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/yarn/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'RESOURCEMANAGER'
        , ->
          @call 'ryba-ambari-takeover/ranger/plugins/yarn/install', options
      commands:
        'install':
          'ryba-ambari-takeover/ranger/plugins/yarn/install'
