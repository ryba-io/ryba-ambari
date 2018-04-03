
# Hadoop KMS

Hadoop KMS is a cryptographic key management server based on Hadoop’s
KeyProvider API.

It provides a client and a server components which communicate over HTTP using a
REST API.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        java: module: 'masson/commons/java', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
      configure:
        'ryba-ambari-takeover/hadoop/kms/configure'
      commands:
        'check':
          'ryba-ambari-takeover/hadoop/kms/check'
        'install': [
          'ryba-ambari-takeover/hadoop/kms/install'
          'ryba-ambari-takeover/hadoop/kms/check'
        ]
