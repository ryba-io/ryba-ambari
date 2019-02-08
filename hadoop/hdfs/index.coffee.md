
# Hadoop takeover

This modules aims at installing HDFS service with ambari.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        hdfs: module: 'ryba-ambari-takeover/hadoop/hdfs'
      configure: 'ryba-ambari-takeover/hadoop/hdfs/configure'
