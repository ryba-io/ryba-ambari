
# Ambari Logsearch Feeder

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_infra_service: module: 'ryba-ambari-takeover/ambari_infra/service', required: true
      configure:
        'ryba-ambari-takeover/ambari_infra/instance/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/ambari_infra/instance/install'
          # 'ryba-ambari-takeover/ambari_infra/instance/start'
        ]
        'start': 'ryba-ambari-takeover/ambari_infra/instance/start'
        'stop': 'ryba-ambari-takeover/ambari_infra/instance/stop'

[Ambari-server]: http://ambari.apache.org
