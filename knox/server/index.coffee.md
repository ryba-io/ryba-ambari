
# Knox

The Apache Knox Gateway is a REST API gateway for interacting with Apache Hadoop
clusters. The gateway provides a single access point for all REST interactions
with Hadoop clusters.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        sssd: module: 'masson/core/sssd', local: true
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true, implicit: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        # hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        # hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        # hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client'
        httpfs: module: 'ryba-ambari-takeover/hadoop/httpfs'
        # yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        # yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        # yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        # mapred_jhs: module: 'ryba-ambari-takeover/hadoop/mapred_jhs'
        # hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        # hive_webhcat: module: 'ryba-ambari-takeover/hive/webhcat'
        oozie: module: 'ryba-ambari-takeover/oozie/service'
        # hbase_rest: module: 'ryba-ambari-takeover/hbase/rest'
        knox_server: module: 'ryba-ambari-takeover/knox/server'
        ranger_admin: module: 'ryba-ambari-takeover/ranger/hdpadmin', single: true
        log4j: module: 'ryba/ambari/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
        knox: module: 'ryba-ambari-takeover/knox/service', required: true
      configure:
        'ryba-ambari-takeover/knox/server/configure'
      commands:
        install: [
          'ryba-ambari-takeover/knox/server/install'
          'ryba-ambari-takeover/knox/server/start'
          'ryba-ambari-takeover/knox/server/check'
        ]
        check:
          'ryba-ambari-takeover/knox/server/check'
        start:
          'ryba-ambari-takeover/knox/server/start'
        stop:
          'ryba-ambari-takeover/knox/server/stop'
        status:
          'ryba-ambari-takeover/knox/server/status'