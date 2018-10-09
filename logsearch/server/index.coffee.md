
# Ambari Logsearch Server

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        logsearch_server: module: 'ryba-ambari-takeover/logsearch/server', required: true
        logsearch_service: module: 'ryba-ambari-takeover/logsearch/service', required: true
        ambari_infra_instance: module: 'ryba-ambari-takeover/ambari_infra/service'
      configure:
        'ryba-ambari-takeover/logsearch/server/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/logsearch/server/install'
          'ryba-ambari-takeover/logsearch/server/start'
        ]
        'start': 'ryba-ambari-takeover/logsearch/server/start'
        'stop': 'ryba-ambari-takeover/logsearch/server/stop'

[Ambari-server]: http://ambari.apache.org
