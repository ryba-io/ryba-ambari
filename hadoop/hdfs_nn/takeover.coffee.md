
# Ambari HDFS NAMENODE Takeover

    module.exports = header: 'HDFS Namenode Takeover', handler: (options) ->

## Register

      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_update'], "ryba-ambari-actions/lib/hosts/component_update"

### Add HDFS Service


      @ambari.services.add
        header: 'HDFS Service'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'HDFS'


