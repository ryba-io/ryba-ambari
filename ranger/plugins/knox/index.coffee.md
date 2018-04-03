# Ranger Knox Plugin

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: use: true, module: 'ryba-ambari-takeover/hadoop/core', local: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        knox_server: module: 'ryba-ambari-takeover/knox/server', local: true
        knox: module: 'ryba-ambari-takeover/knox/service'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true, required: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
      configure:
        'ryba-ambari-takeover/ranger/plugins/knox/configure'
      plugin: (options) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'KNOX_GATEWAY'
        , ->
          delete options.original.type
          delete options.original.handler
          delete options.original.argument
          delete options.original.store
          @call 'ryba-ambari-takeover/ranger/plugins/knox/install', options.original
      commands:
        install:
          'ryba-ambari-takeover/ranger/plugins/knox/install'
