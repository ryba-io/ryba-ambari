
# Tez

[Apache Tez][tez] is aimed at building an application framework which allows for
a complex directed-acyclic-graph of tasks for processing data. It is currently
built atop Apache Hadoop YARN.

## Commands

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true
        # httpd: module: 'masson/commons/httpd', local: true #tez-ui
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, auto: true, implicit: true
        hive: module: 'ryba-ambari-takeover/hive/service', required: true #Tez can be viewed as a component depending on all the hive service
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', required: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm', required: true
        # yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts', required: true #tez-ui
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client', local: true, auto: true, implicit: true
        ambari_server: module: 'ryba-ambari-takeover/server', required: true, single: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/tez/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/tez/install'
          'ryba-ambari-takeover/tez/check'
        ]
        'check':
          'ryba-ambari-takeover/tez/check'

[tez]: http://tez.apache.org/
[instructions]: (http://docs.hortonworks.com/HDPDocuments/HDP2/HDP-2.2.0/HDP_Man_Install_v22/index.html#Item1.8.4)
