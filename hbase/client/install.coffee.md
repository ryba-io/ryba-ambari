
# HBase Client Install

Install the HBase client package and configure it with secured access.

    module.exports =  header: 'HBase Client Install', handler: (options) ->

## Register

      @registry.register 'hconfigure', 'ryba/lib/hconfigure'
      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"

## Packages

      @service
        name: 'hbase'
      @hdp_select
        name: 'hbase-client'

# ## Zookeeper JAAS
# 
# JAAS configuration files for zookeeper to be deployed on the HBase Master,
# RegionServer, and HBase client host machines.
# 
#       @file.jaas
#         header: 'Zookeeper JAAS'
#         target: "#{options.conf_dir}/hbase-client.jaas"
#         content: Client:
#           useTicketCache: 'true'
#         uid: options.user.name
#         gid: options.group.name
#         mode: 0o644


# Install Component

      @ambari.hosts.component_wait
        header: 'HBASE_CLIENT WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'HBASE_CLIENT INSTALL'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HBASE_CLIENT'
        hostname: options.fqdn
