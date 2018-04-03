
# Pig

[Apache Pig](https://pig.apache.org/) is a platform for analyzing large data sets that consists of a
high-level language for expressing data analysis programs, coupled with
infrastructure for evaluating these programs. The salient property of Pig
programs is that their structure is amenable to substantial parallelization,
which in turns enables them to handle very large data sets.

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', local: true, required: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm', required: true
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client', local: true, auto: true, implicit: true
        mapred_client: module: 'ryba-ambari-takeover/hadoop/mapred_client', local: true, auto: true, implicit: true
        hive_client: module: 'ryba-ambari-takeover/hive/client', local: true, required: true # In case pig is run through hcat
        ranger_admin: module: 'ryba-ambari-takeover/ranger/admin'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/pig/configure'
      commands:
        'check':
          'ryba-ambari-takeover/pig/check'
        'install': [
          'ryba-ambari-takeover/pig/install'
          'ryba-ambari-takeover/pig/check'
        ]
