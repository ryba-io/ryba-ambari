
# Oozie Client

Oozie is a server based Workflow Engine specialized in running workflow jobs
with actions that run Hadoop Map/Reduce and Pig jobs.

The Oozie server installation includes the Oozie client. The Oozie client should
be installed in remote machines only.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        krb5_client: implicit: true, module: 'masson/core/krb5_client'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hiveserver2'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client', local: true, auto: true, implicit: true
        mapred_client: module: 'ryba-ambari-takeover/hadoop/mapred_client', local: true, auto: true, implicit: true
        hive_client: module: 'ryba-ambari-takeover/hive/client', local: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
        oozie_server: module: 'ryba-ambari-takeover/oozie/server'
        oozie: module: 'ryba-ambari-takeover/oozie/service', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure: 'ryba-ambari-takeover/oozie/client/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/oozie/client/install'
          'ryba-ambari-takeover/oozie/client/check'
        ]
        'check':
          'ryba-ambari-takeover/oozie/client/check'
        'deploy':
          'ryba-ambari-takeover/oozie/client/install'
