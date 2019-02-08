
# Ambari Metrics

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        metrics_service: module: 'ryba-ambari-takeover/metrics/service', required: true
      configure:
        'ryba-ambari-takeover/metrics/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/metrics/service/install'
        ]

[Ambari-server]: http://ambari.apache.org
