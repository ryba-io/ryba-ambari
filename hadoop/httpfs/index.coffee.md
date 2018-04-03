
# HDFS HttpFS

HttpFS is a server that provides a REST HTTP gateway supporting all HDFS File
System operations (read and write). And it is inteoperable with the webhdfs REST
HTTP API.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true
        httpfs: module: 'ryba-ambari-takeover/hadoop/httpfs'
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', local: true, required: true
        log4j: module: 'ryba/log4j', local: true
      configure:
        'ryba-ambari-takeover/hadoop/httpfs/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hadoop/httpfs/check'
        'install': [
          'ryba-ambari-takeover/hadoop/httpfs/install'
          'ryba-ambari-takeover/hadoop/httpfs/start'
          'ryba-ambari-takeover/hadoop/httpfs/check'
        ]
        'start':
          'ryba-ambari-takeover/hadoop/httpfs/start'
        'stop':
          'ryba-ambari-takeover/hadoop/httpfs/stop'
        'status':
          'ryba-ambari-takeover/hadoop/httpfs/status'
