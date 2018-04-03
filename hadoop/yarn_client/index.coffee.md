
# YARN Client

The [Hadoop YARN Client](http://hadoop.apache.org/docs/current/hadoop-yarn/hadoop-yarn-site/WebServicesIntro.html) web service REST APIs are a set of URI resources that give access to the cluster, nodes, applications, and application historical information.
The URI resources are grouped into APIs based on the type of information returned. Some URI resources return collections while others return singletons.

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', required: true
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', required: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm', required: true
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hadoop/yarn_client/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hadoop/yarn_client/check'
        'install': [
          'ryba-ambari-takeover/hadoop/yarn_client/install'
          'ryba-ambari-takeover/hadoop/yarn_client/check'
        ]
        'report': [
          'masson/bootstrap/report'
          'ryba-ambari-takeover/hadoop/yarn_client/report'
        ]
