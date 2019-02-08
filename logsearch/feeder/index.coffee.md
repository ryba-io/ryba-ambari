
# Ambari Logsearch Feeder

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        logsearch_service: module: 'ryba-ambari-takeover/logsearch/service', required: true
        logsearch_server: module: 'ryba-ambari-takeover/logsearch/server', required: true
        logsearch_feeder: module: 'ryba-ambari-takeover/logsearch/feeder', required: true
      configure:
        'ryba-ambari-takeover/logsearch/feeder/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/logsearch/feeder/install'
        ]
        'start': 'ryba-ambari-takeover/logsearch/feeder/start'
        'stop': 'ryba-ambari-takeover/logsearch/feeder/stop'

[Ambari-server]: http://ambari.apache.org
