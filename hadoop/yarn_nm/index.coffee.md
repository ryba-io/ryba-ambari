
# YARN NodeManager

[The NodeManager](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/YARN.htm) (NM) is YARN’s per-node agent,
and takes care of the individual
computing nodes in a Hadoop cluster. This includes keeping up-to date with the
ResourceManager (RM), overseeing containers’ life-cycle management; monitoring
resource usage (memory, CPU) of individual containers, tracking node-health,
log’s management and auxiliary services which may be exploited by different YARN
applications.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        cgroups: module: 'masson/core/cgroups', local: true, required: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', required: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        ranger_admin: module: 'ryba/ranger/admin'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        metrics: module: 'ryba/metrics', local: true
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hadoop/yarn_nm/configure'
      commands:
        # 'backup':
        #   'ryba-ambari-takeover/hadoop/yarn_nm/backup'
        'check':
          'ryba-ambari-takeover/hadoop/yarn_nm/check'
        'install': [
          'masson/core/info'
          'ryba-ambari-takeover/hadoop/yarn_nm/install'
          'ryba-ambari-takeover/hadoop/yarn_nm/start'
          'ryba-ambari-takeover/hadoop/yarn_nm/check'
        ]
        'report': [
          'masson/bootstrap/report'
          'ryba-ambari-takeover/hadoop/yarn_nm/report'
        ]
        'start':
          'ryba-ambari-takeover/hadoop/yarn_nm/start'
        'status':
          'ryba-ambari-takeover/hadoop/yarn_nm/status'
        'stop':
          'ryba-ambari-takeover/hadoop/yarn_nm/stop'
        'takeover': [
          'ryba-ambari-takeover/hadoop/yarn_nm/wait'
          'ryba-ambari-takeover/hadoop/yarn_nm/install'
          'ryba-ambari-takeover/hadoop/yarn_nm/takeover'
          'ryba-ambari-takeover/hadoop/yarn_nm/start'
          'ryba-ambari-takeover/hadoop/yarn_nm/wait'
          'ryba-ambari-takeover/hadoop/yarn_nm/check'
        ]