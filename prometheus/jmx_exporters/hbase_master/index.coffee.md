
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        hbase_master: module: 'ryba-ambari-takeover/hbase/master', local: true, required: true
        hbase_service: module: 'ryba-ambari-takeover/hbase/service', required: true
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master'
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/configure'
      plugin: (options) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'HBASE_MASTER'
        , ->
          delete options.original.type
          delete options.original.handler
          delete options.original.argument
          delete options.original.store
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/password.coffee.md', options.original
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/password'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hbase_master/prepare'
        ]
