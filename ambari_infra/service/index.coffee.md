
# Ambari Logsearch

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_infra_service: module: 'ryba-ambari-takeover/ambari_infra/service', required: true
      configure:
        'ryba-ambari-takeover/ambari_infra/service/configure'


[Ambari-server]: http://ambari.apache.org
