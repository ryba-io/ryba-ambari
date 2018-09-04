
# Zookeeper Client Install

    module.exports = header: 'ZooKeeper Client Ambari Install', handler: ({options}) ->

## Register

      @registry.register 'hdp_select', 'ryba/lib/hdp_select'
      @registry.register ['file', 'jaas'], 'ryba/lib/file_jaas'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','cluster','node_add'], 'ryba-ambari-actions/lib/cluster/node_add'
      @registry.register ['ambari','hosts','add'], 'ryba-ambari-actions/lib/hosts/add'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"

## Ambari Service & components

      @ambari.services.wait
        header: 'WAIT Service'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'ZOOKEEPER'

      @ambari.services.component_add
        header: 'ADD COMPONENT TO SERVICE'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        service_name: 'ZOOKEEPER'

      @ambari.hosts.component_add
        header: 'ADD COMPONENT TO HOST'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'set Installed'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'ZOOKEEPER_CLIENT'
        hostname: options.fqdn
