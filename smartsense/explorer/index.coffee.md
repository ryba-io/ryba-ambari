
# Ambari Logsearch Server

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        smartsense_service: module: 'ryba-ambari-takeover/smartsense/service', required: true
      configure:
        'ryba-ambari-takeover/smartsense/explorer/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/smartsense/explorer/install'
        ]

[Ambari-server]: http://ambari.apache.org
