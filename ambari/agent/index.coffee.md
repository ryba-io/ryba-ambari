
# Ambari Client

[Ambari-agent][Ambari-agent-install] on hosts enables the ambari server to be
aware of the  hosts where Hadoop will be deployed. The Ambari Server must be 
installed before the agent registration.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true, recommanded: true
        ambari_server: module: 'ryba-ambari-takeover/ambari/server', required: true
        ambari_repo: module: 'ryba-ambari-takeover/ambari/repo', local: true, implicit: true
        ambari_agent: module: 'ryba-ambari-takeover/ambari/agent'
        local_agent: module: 'ryba-ambari-takeover/agent', local: true, required: true
      configure:
        'ryba-ambari-takeover/ambari/agent/configure'
      commands:
        'install': [
          'ryba-ambari-takeover/ambari/agent/install'
          'ryba-ambari-takeover/ambari/agent/start'
        ]
        'start':
          'ryba-ambari-takeover/ambari/agent/start'
        'stop':
          'ryba-ambari-takeover/ambari/agent/stop'

[Ambari-agent-install]: https://cwiki.apache.org/confluence/display/AMBARI/Installing+ambari-agent+on+target+hosts
