
# Apache Spark Install

[Spark Installation][Spark-install] following hortonworks guidelines to install
Spark requires HDFS and Yarn. Install spark in Yarn cluster mode.

Resources:

[Tips and Tricks from Altic Scale][https://www.altiscale.com/blog/tips-and-tricks-for-running-spark-on-hadoop-part-2-2/)   

    module.exports = header: 'Ambari Spark History Server Install', handler: ({options}) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdfs_mkdir', 'ryba/lib/hdfs_mkdir'
      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Install Component

      @ambari.hosts.component_wait
        header: 'SPARK_JOBHISTORYSERVER Wait'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK_JOBHISTORYSERVER'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'SPARK_JOBHISTORYSERVER Install'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'SPARK_JOBHISTORYSERVER'
        hostname: options.fqdn

## Dependencies

    mkcmd = require 'ryba/lib/mkcmd'
    quote = require 'regexp-quote'
    string = require 'nikita/lib/misc/string'

[spark-conf]:https://spark.apache.org/docs/latest/configuration.html
