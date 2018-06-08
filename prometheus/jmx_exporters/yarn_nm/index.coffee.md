
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        yarn_nm: module: 'ryba-ambari-takeover/hadoop/yarn_nm', local: true, required: true
        yarn_service: module: 'ryba-ambari-takeover/hadoop/yarn'
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/configure'
      plugin: (options) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'NODEMANAGER'
        , ->
          delete options.original.type
          delete options.original.handler
          delete options.original.argument
          delete options.original.store
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/password.coffee.md', options.original
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/password'
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_nm/prepare'
        ]
