
# Graceful Stop For HBase regionserver

    module.exports = header: 'Graceful Stop HBase Regionserver', handler: ({options}) ->

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'
      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Steps

      
      @system.execute
        cmd: mkcmd.hbase options.admin, """
        /usr/hdp/current/hbase-regionserver/bin/graceful_stop.sh --config /etc/hbase-regionserver/conf --maxthreads 32 #{options.fqdn}
        """
      @ambari.hosts.component_stop
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_REGIONSERVER'
        hostname: options.fqdn
  
    mkcmd = require 'ryba/lib/mkcmd'
