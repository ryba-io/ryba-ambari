
# YARN Timeline Server

The [Yarn Timeline Server][ts] store and retrieve current as well as historic
information for the applications running inside YARN.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', auto: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', required: true
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', required: true
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn'
        # yarn_client: 'ryba-ambari-takeover/hadoop/yarn_client'
      configure:
        'ryba-ambari-takeover/hadoop/yarn_ts/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hadoop/yarn_ts/check'
        'install': [
          'ryba-ambari-takeover/hadoop/yarn_ts/install'
          'ryba-ambari-takeover/hadoop/yarn_ts/start'
          'ryba-ambari-takeover/hadoop/yarn_ts/check'
        ]
        'start':
          'ryba-ambari-takeover/hadoop/yarn_ts/start'
        'status':
          'ryba-ambari-takeover/hadoop/yarn_ts/status'
        'stop':
          'ryba-ambari-takeover/hadoop/yarn_ts/stop'

[ts]: http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/TimelineServer.html
