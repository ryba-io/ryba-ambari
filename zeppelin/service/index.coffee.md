
# Zeppelin

    module.exports =
      deps:
        krb5: module: 'masson/core/krb5_client', local: true
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        zeppelin_service: module: 'ryba-ambari-takeover/zeppelin/service'
        spark_livy_server: module: 'ryba-ambari-takeover/spark/livy'
      configure:
        'ryba-ambari-takeover/zeppelin/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/zeppelin/service/install'
        ]

[Ambari-server]: http://ambari.apache.org
