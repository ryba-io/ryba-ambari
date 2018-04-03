
# Ambari Metrics

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent', required: true
        metrics_service: module: 'ryba-ambari-takeover/metrics/service', required: true
      configure:
        'ryba-ambari-takeover/metrics/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/metrics/service/install'
        ]

[Ambari-server]: http://ambari.apache.org
