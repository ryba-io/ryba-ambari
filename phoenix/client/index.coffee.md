
# Phoenix

Apache Phoenix is a relational database layer over HBase delivered as a client-embedded
JDBC driver targeting low latency queries over HBase data. Apache Phoenix takes
your SQL query, compiles it into a series of HBase scans, and orchestrates the
running of those scans to produce regular JDBC result sets.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local:true, implicit: true
        test_user: module: 'ryba/commons/test_user', local: true, auto: true
        hbase: module: 'ryba-ambari-takeover/hbase/service'
        hbase_master: module: 'ryba-ambari-takeover/hbase/master'
        hbase_master_local: module: 'ryba-ambari-takeover/hbase/master', local: true
        hbase_regionserver: module: 'ryba-ambari-takeover/hbase/regionserver'
        hbase_regionserver_local: module: 'ryba-ambari-takeover/hbase/regionserver', local: true
        hbase_client: module: 'ryba-ambari-takeover/hbase/client'
        hbase_client_local: module: 'ryba-ambari-takeover/hbase/client', local: true
      configure:
        'ryba-ambari-takeover/phoenix/client/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_MASTER'
        , ->
          @call 'ryba-ambari-takeover/phoenix/client/install', options
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_REGIONSERVER'
        , ->
          @call 'ryba-ambari-takeover/phoenix/client/install', options
      commands:
        'install': [
          'ryba/phoenix/client/install'
          'ryba/phoenix/client/init'
          'ryba/phoenix/client/check'
        ]
