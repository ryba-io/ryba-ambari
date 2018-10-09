
# Ambari Logsearch

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_service: module: 'ryba/ambari/server', required: true
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent', required: true
        ambari_infra_service: module: 'ryba-ambari-takeover/ambari_infra/service', required: true
      configure:
        'ryba-ambari-takeover/ambari_infra/service/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/ambari_infra/service/install'
        ]

[Ambari-server]: http://ambari.apache.org
