
# MapReduce Client

MapReduce is the key algorithm that the Hadoop MapReduce engine uses to distribute work around a cluster.
The key aspect of the MapReduce algorithm is that if every Map and Reduce is independent of all other ongoing Maps and Reduces,
then the operation can be run in parallel on different keys and lists of data. On a large cluster of machines, you can go one step further, and run the Map operations on servers where the data lives.
Rather than copy the data over the network to the program, you push out the program to the machines.
The output list can then be saved to the distributed filesystem, and the reducers run to merge the results. Again, it may be possible to run these in parallel, each reducing different keys.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        java: module: 'masson/commons/java', local: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hdfs_client: module: 'ryba-ambari-takeover/hadoop/hdfs_client', required: true
        yarn_client: module: 'ryba-ambari-takeover/hadoop/yarn_client', required: true
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', required: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm', required: true
        yarn_ts: module: 'ryba-ambari-takeover/hadoop/yarn_ts', required: true, single: true
        mapred_jhs: module: 'ryba-ambari-takeover/hadoop/mapred_jhs', single: true
        yarn: module: 'ryba-ambari-takeover/hadoop/yarn', required: true
        mapreduce: module: 'ryba-ambari-takeover/hadoop/mapreduce', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
      configure:
        'ryba-ambari-takeover/hadoop/mapred_client/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hadoop/mapred_client/check'
        'report': [
          'masson/bootstrap/report'
          'ryba-ambari-takeover/hadoop/mapred_client/report'
        ]
        'install': [
          'ryba-ambari-takeover/hadoop/mapred_client/install'
          'ryba-ambari-takeover/hadoop/mapred_client/check'
        ]
        'deploy': 'ryba-ambari-takeover/hadoop/mapred_client/install'
