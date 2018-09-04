
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        hbase_rs: module: 'ryba-ambari-takeover/hbase/regionserver', local: true, required: true
        hbase_service: module: 'ryba-ambari-takeover/hbase/service', required: true
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_REGIONSERVER'
        , ->
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/password.coffee.md', options
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/password'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_rs/prepare'
        ]
