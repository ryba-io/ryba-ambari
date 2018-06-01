
# Graceful Stop For HBase regionserver

    module.exports = header: 'Graceful Stop HBase Regionserver', handler: (options) ->

      @registry.register ['ambari','hosts','component_status'], 'ryba-ambari-actions/lib/hosts/component_status'
      @registry.register ['ambari','hosts','component_stop'], 'ryba-ambari-actions/lib/hosts/component_stop'

## Steps

      
      @system.execute
        cmd: mkcmd.hbase options.admin,"""
        su -l hbase -c "kinit -kt /etc/security/keytabs/hbase.service.keytab -p hbase/#{options.fqdn} ; /usr/hdp/current/hbase-regionserver/bin/graceful_stop.sh --config /etc/hbase/conf --maxthreads 64 #{options.fqdn} "
        """

      # @ambari.hosts.component_stop
      #   url: options.ambari_url
      #   username: 'admin'
      #   password: options.ambari_admin_password
      #   cluster_name: options.cluster_name
      #   name: 'HBASE_REGIONSERVER'
      #   hostname: options.fqdn

      @ambari.hosts.component_status
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HBASE_REGIONSERVER'
        status: 'INSTALLED'
        hostname: options.fqdn
  
    mkcmd = require 'ryba/lib/mkcmd'
