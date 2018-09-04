
# JMX Exporter

JMX to Prometheus exporter.
A Collector that can configurably scrape and expose mBeans of a JMX target. 
It meant to be run as a Java Agent, exposing an HTTP server and scraping the local JVM.

    module.exports =
      deps:
        ssl: module: 'masson/core/ssl', local: true
        java: module: 'masson/commons/java', local: true, required: true
        iptables: module: 'masson/core/iptables', local: true
        hdfs_dn: module: 'ryba-ambari-takeover/hadoop/hdfs_dn', local: true, required: true
        hdfs_service: module: 'ryba-ambari-takeover/hadoop/hdfs', required: true
        hadoop_core: module: 'ryba-ambari-takeover/hadoop/core', local: true
        jmx_exporter: module: 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn'
        prometheus_monitor: module: 'ryba/prometheus/monitor', required: true
      configure: 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/configure'
      plugin: ({options}) ->
        @before
          type: ['ambari', 'hosts', 'component_start']
          name: 'DATANODE'
        , ->
          @call 'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/password.coffee.md', options
      commands:
        install: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/password'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/install'
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/start'
        ]
        start : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/start'
        ]
        stop : [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/stop'
        ]
        prepare: [
          'ryba-ambari-takeover/prometheus/jmx_exporters/hdfs_dn/prepare'
        ]
