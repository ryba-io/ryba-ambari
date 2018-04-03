
# Ambari Client

[Ambari-agent][Ambari-agent-install] on hosts enables the ambari server to be
aware of the  hosts where Hadoop will be deployed. The Ambari Server must be 
installed before the agent registration.

    module.exports =
      deps:
        krb5_client: module: 'masson/core/krb5_client', local: true
        ambari_server: module: 'ryba/ambari/server', required: true
        ambari_server_takeover: module: 'ryba-ambari-takeover/server', required: true
        ambari_agent: module: 'ryba/ambari/agent', local: true, required: true
      configure:
        'ryba-ambari-takeover/agent/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/agent/install'
        ]

[Ambari-agent-install]: https://cwiki.apache.org/confluence/display/AMBARI/Installing+ambari-agent+on+target+hosts
