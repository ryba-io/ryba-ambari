
# Hive & HCatolog Client

[Hive Client](https://cwiki.apache.org/confluence/display/Hive/HiveClient) is the application that you use in order to administer, use Hive.
Once installed you can type hive in a prompt and the hive client shell wil launch directly.

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client'
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client'
        mapred_client: 'ryba-ambari-takeover/hadoop/mapred_client'
        tez: module: 'ryba-ambari-takeover/tez', local: true, auto: true, implicit: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        phoenix_client: module: 'ryba/phoenix/client', local: true
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        ranger_hdfs: module: 'ryba-ambari-takeover/ranger/plugins/hdfs'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hive/client/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hive/client/install'
        ]
