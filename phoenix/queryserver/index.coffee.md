
# Phoenix

Apache Phoenix is a relational database layer over HBase delivered as a client-embedded
JDBC driver targeting low latency queries over HBase data. Apache Phoenix takes
your SQL query, compiles it into a series of HBase scans, and orchestrates the
running of those scans to produce regular JDBC result sets.

    module.exports =
      deps:
        iptables: module: 'masson/core/iptables', local: true
        java: module: 'masson/commons/java', local:true, implicit: true
        krb5_client: module: 'masson/core/krb5_client', local: true, required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true, auto: true, implicit: true
        hbase_client: module: 'ryba-ambari-takeover/hbase/client'
        phoenix_client: module: 'ryba-ambari-takeover/phoenix/client'
        ambari_server: module: 'ryba-ambari-takeover/server', single: true, required: true
        ambari_agent: module: 'ryba-ambari-takeover/agent'
      configure:
        'ryba-ambari-takeover/phoenix/queryserver/configure'
      commands:
        install: [
          'ryba-ambari-takeover/phoenix/queryserver/install'
          'ryba-ambari-takeover/phoenix/queryserver/start'
          'ryba-ambari-takeover/phoenix/queryserver/check'
        ]
        check:
          'ryba-ambari-takeover/phoenix/queryserver/check'
        status:
          'ryba-ambari-takeover/phoenix/queryserver/status'
        start:
          'ryba-ambari-takeover/phoenix/queryserver/start'
        stop:
          'ryba-ambari-takeover/phoenix/queryserver/stop'
