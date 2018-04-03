
# Oozie Server

[Oozie Server][Oozie] is a server based Workflow Engine specialized in running workflow jobs.
Workflows are basically collections of actions.
These actions can be  Hadoop Map/Reduce jobs, Pig jobs arranged in a control dependency DAG (Direct Acyclic Graph).
Please check Oozie page

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        mapred_jhs: module: 'ryba-ambari-takeover/hadoop/mapred_jhs'
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'
        hive_hcatalog: module: 'ryba-ambari-takeover/hive/hcatalog'
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        hive_webhcat: module: 'ryba-ambari-takeover/hive/webhcat'
        # spark_client: module: 'ryba/spark/client', local: true, auto: true, implicit: true
        oozie: module: 'ryba-ambari-takeover/oozie/service', required: true
        oozie_server: module: 'ryba-ambari-takeover/oozie/server'
        log4j: module: 'ryba/log4j', local: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure: 'ryba-ambari-takeover/oozie/server/configure'
      commands:
        backup:
          'ryba-ambari-takeover/oozie/server/backup'
        install: [
          'ryba-ambari-takeover/oozie/server/install'
          'ryba-ambari-takeover/oozie/server/start'
          'ryba-ambari-takeover/oozie/server/check'
        ]
        start:
          'ryba-ambari-takeover/oozie/server/start'
        status:
          'ryba-ambari-takeover/oozie/server/status'
        stop:
          'ryba-ambari-takeover/oozie/server/stop'

[Oozie]: https://oozie.apache.org/docs/3.1.3-incubating/index.html
