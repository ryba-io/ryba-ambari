
# HBase Client

Install the [HBase client](https://hbase.apache.org/apidocs/org/apache/hadoop/hbase/client/package-summary.html) package and configure it with secured access.
you have to use it for administering HBase, create and drop tables, list and alter tables.
Client code accessing a cluster finds the cluster by querying ZooKeeper.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        mapred_client: module: 'ryba-ambari-takeover/hadoop/mapred_client', required: true
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master', required: true
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver', required: true
        ranger_admin: module: 'ryba/ranger/admin', single: true
        ranger_hbase: module: 'ryba/ranger/plugins/hbase'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hbase/client/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/hbase/client/install'
          'ryba-ambari-takeover/hbase/client/replication'
          'ryba-ambari-takeover/hbase/client/check'
        ]
        'check':
          'ryba-ambari-takeover/hbase/client/check'
        'deploy':
          'ryba-ambari-takeover/hbase/client/install'
