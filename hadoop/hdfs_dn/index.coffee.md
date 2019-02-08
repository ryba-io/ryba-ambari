
# Hadoop HDFS DataNode

A [DataNode](http://wiki.apache.org/hadoop/DataNode) manages the storage attached
to the node it run on. There are usually one DataNode per node in the cluster.
HDFS exposes a file system namespace and allows user data to be stored in files.
Internally, a file is split into one or more blocks and these blocks are stored
in a set of DataNodes. The DataNodes also perform block creation, deletion, and
replication upon instruction from the NameNode.

To provide a fast failover in a Higth Availabity (HA) enrironment, it is
necessary that the Standby node have up-to-date information regarding the
location of blocks in the cluster. In order to achieve this, the DataNodes are
configured with the location of both NameNodes, and send block location
information and heartbeats to both.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn'
        metrics: module: 'ryba/metrics', local: true
        log4j: module: 'ryba/log4j', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
      configure:
        'ryba-ambari-takeover/hadoop/hdfs_dn/configure'
      commands:
        'start': 'ryba-ambari-takeover/hadoop/hdfs_dn/start'
        'stop': 'ryba-ambari-takeover/hadoop/hdfs_dn/stop'
        'check': 'ryba-ambari-takeover/hadoop/hdfs_dn/check'
