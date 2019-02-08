
# HBase Rest Gateway
Stargate is the name of the REST server bundled with HBase.
The [REST Server](http://wiki.apache.org/hadoop/Hbase/Stargate) is a daemon which enables other application to request HBASE database via http.
Of course we deploy the secured version of the configuration of this API.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/core', required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, required: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master', required: true
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver', required: true
        hbase_client: module: 'ryba-ambari-takeover/hbase/client', local: true
        hbase_rest: module: 'ryba-ambari-takeover/hbase/rest'
        hbase: module: 'ryba-ambari-takeover/hbase/service', required: true
        ranger_admin: module: 'ryba/ranger/admin', single: true
        ranger_hbase: module: 'ryba/ranger/plugins/hbase'
      configure:
        'ryba-ambari-takeover/hbase/rest/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hbase/rest/check'
        'install': [
          'ryba-ambari-takeover/hbase/rest/install'
          'ryba-ambari-takeover/hbase/rest/start'
          'ryba-ambari-takeover/hbase/rest/check'
        ]
        'start':
          'ryba-ambari-takeover/hbase/rest/start'
        'status':
          'ryba-ambari-takeover/hbase/rest/status'
        'stop':
          'ryba-ambari-takeover/hbase/rest/stop'
