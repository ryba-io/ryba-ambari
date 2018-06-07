
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        hdfs_nn: module: 'ryba-ambari-takeover/hadoop/hdfs_nn', local: true, required: true
        hdfs_service: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn'
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/configure'
      plugin: (options) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'NAMENODE'
        , ->
          delete options.original.type
          delete options.original.handler
          delete options.original.argument
          delete options.original.store
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/password.coffee.md', options.original
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_nn/prepare'
        ]
