
# Ambari HDFS HDFS_CLIENT Takeover

    module.exports = header: 'HDFS Client Takeover', handler: (options) ->

## Register

      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"

### Wait HDFS Service


      @ambari.services.wait
        header: 'HDFS Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HDFS'


      @ambari.services.component_add
        header: 'HDFS_CLIENT'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HDFS_CLIENT'
        service_name: 'HDFS'


### HDFS_CLIENT COMPONENT

      @ambari.hosts.component_add
        header: 'HDFS_CLIENT ADD'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HDFS_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_update
        header: 'HDFS_CLIENT UPDATE'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'HDFS_CLIENT'
        hostname: options.fqdn
        properties: 'HostRoles': state: 'INSTALLED'

