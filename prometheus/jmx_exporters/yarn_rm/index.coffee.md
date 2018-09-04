
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        yarn_rm: module: 'ryba-ambari-takeover/hadoop/yarn_rm', local: true, required: true
        yarn_service: module: 'ryba-ambari-takeover/hadoop/yarn'
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'RESOURCEMANAGER'
        , ->
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/password.coffee.md', options
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/password'
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/yarn_rm/prepare'
        ]
