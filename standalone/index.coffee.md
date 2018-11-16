
# Ambari Server

[Ambari-server][Ambari-server] is the master host for ambari software.
Once logged into the ambari server host, the administrator can  provision, 
manage and monitor a Hadoop cluster.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        db_admin: module: 'ryba/commons/db_admin', local: true, auto: true, implicit: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        ambari_repo: module: 'ryba/ambari/repo', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn'
        mapred: module: 'ryba-ambari-takeover/hadoop/mapreduc'
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts'
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm'
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm'
        hive_server2: module: 'ryba-ambari-takeover/hive/server2'
        ranger_hive: module: 'ryba-ambari-takeover/ranger/plugins/hive'
        oozie_server: module: 'ryba-ambari-takeover/oozie/server'
        ambari_standalone: module: 'ryba-ambari-takeover/standalone'
      configure: 'ryba-ambari-takeover/standalone/configure'
      commands:
        'ambari_blueprint': 'ryba-ambari-takeover/standalone/blueprint'
        'check': 'ryba-ambari-takeover/standalone/check'
        'install': [
          'ryba-ambari-takeover/standalone/install'
          'ryba-ambari-takeover/standalone/start'
          'ryba-ambari-takeover/standalone/check'
          'ryba-ambari-takeover/views'
        ]
        'start': 'ryba-ambari-takeover/standalone/start'
        'stop': 'ryba-ambari-takeover/standalone/stop'

[Ambari-server]: http://ambari.apache.org
