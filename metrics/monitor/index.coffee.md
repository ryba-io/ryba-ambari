
# Ambari Metrics Monitor

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        metrics_service: module: 'ryba-ambari-takeover/metrics/service', required: true
        metrics_collector: module: 'ryba-ambari-takeover/metrics/collector', required: true
      configure:
        'ryba-ambari-takeover/metrics/monitor/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/metrics/monitor/install'
          'ryba-ambari-takeover/metrics/monitor/start'
        ]
       'start': 'ryba-ambari-takeover/metrics/monitor/start'
       'stop': 'ryba-ambari-takeover/metrics/monitor/stop'

[Ambari-server]: http://ambari.apache.org
