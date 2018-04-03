
# Ambari Metrics Collector

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        ambari_server: module: 'ryba/ambari/server', required: true, single: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        metrics_service: module: 'ryba-ambari-takeover/metrics/service', required: true
      configure:
        'ryba-ambari-takeover/metrics/collector/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/metrics/collector/install'
          'ryba-ambari-takeover/metrics/collector/start'
        ]
       'start': 'ryba-ambari-takeover/metrics/collector/start'
       'stop': 'ryba-ambari-takeover/metrics/collector/stop'

[Ambari-server]: http://ambari.apache.org
