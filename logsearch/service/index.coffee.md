
# Ambari Logsearch

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        logsearch_service: module: 'ryba-ambari-takeover/logsearch/service', required: true
      configure:
        'ryba-ambari-takeover/logsearch/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/logsearch/service/install'
        ]

[Ambari-server]: http://ambari.apache.org
