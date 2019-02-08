
# Ambari Logsearch

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        iptables: module: 'masson/core/iptables', local: true
        smartsense_service: module: 'ryba-ambari-takeover/smartsense/service', required: true
      configure:
        'ryba-ambari-takeover/smartsense/agent/configure'

[Ambari-server]: http://ambari.apache.org
