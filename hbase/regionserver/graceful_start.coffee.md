
# Graceful Stop For HBase regionserver

    module.exports = header: 'Graceful Stop HBase Regionserver', handler: (options) ->

## Steps

      @registry.register ['ambari','hosts','component_start'], 'ryba-ambari-actions/lib/hosts/component_start'
      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'
      @ambari.hosts.component_start
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_REGIONSERVER'
        hostname: options.fqdn
      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_REGIONSERVER'
        status: 'STARTED'
        hostname: options.fqdn

      @system.execute
        cmd: mkcmd.hbase options.admin, """
        echo 'balance_switch true; balancer' | hbase shell
        """
      
    mkcmd = require 'ryba/lib/mkcmd'
