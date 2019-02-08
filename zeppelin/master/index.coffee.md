
# Zeppelin

    module.exports =
      deps:
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true
        zeppelin_service: module: 'ryba-ambari-takeover/zeppelin/service', required: true
      configure:
        'ryba-ambari-takeover/zeppelin/master/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/zeppelin/master/install'
        ]
