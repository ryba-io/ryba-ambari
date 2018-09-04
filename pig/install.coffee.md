
# Pig Install

Learn more about Pig optimization by reading ["Making Pig Fly"][fly].

    module.exports = header: 'Ambari Pig Install', handler: ({options}) ->

## Registry

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"


## Request From Ambari to post default configuration for PIG
      
      @ambari.configs.default
        header: 'PIG Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        installed_services: options.ambari_stack_services
        stack_name: options.stack_name
        stack_version: options.stack_version
        target_services: 'PIG'

## Add PIG Service

      @ambari.services.add
        header: 'PIG Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'PIG'

      @ambari.services.wait
        header: 'PIG Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'PIG'

      @ambari.services.component_add
        header: 'PIG Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'PIG'
        service_name: 'PIG'

## Install Component

      @ambari.hosts.component_add
        header: 'PIG Host Add'
        url: options.ambari_url
        if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'PIG'
        hostname: options.fqdn

      @ambari.hosts.component_wait
        header: 'PIG Wait'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'PIG'
        hostname: options.fqdn

      @ambari.hosts.component_install
        header: 'PIG Install'
        if: options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'PIG'
        hostname: options.fqdn
