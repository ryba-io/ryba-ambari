
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        zookeeper_server: module: 'ryba-ambari-takeover/zookeeper/server', local: true, required: true
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/configure'
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/zookeeper/prepare'
        ]
