
# Zookeeper Client

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        test_user: module: 'ryba/commons/test_user', local: true
        hdp: module: 'ryba/hdp', local: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server'
      configure:
        'ryba-ambari-takeover/zookeeper/client/configure'
      commands:
        'check':
          'ryba-ambari-takeover/zookeeper/client/check'
        'install': [
          'ryba-ambari-takeover/zookeeper/client/install'
          # 'ryba-ambari-takeover/zookeeper/client/check'
        ]
        
