
# Ambari Logsearch Install

    module.exports =  header: 'Ambari Infra Install', handler: ({options}) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','stacks','default'], 'ryba-ambari-actions/lib/stacks/default_informations'
      @registry.register ['ambari','configs','update'], 'ryba-ambari-actions/lib/configs/update'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Upload Default Configuration

      @ambari.configs.default
        header: 'AMBARI_INFRA Configuration'
        url: options.ambari_url
        if: options.post_component and options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'AMBARI_INFRA'

## Add AMBARI_INFRA Service

      @ambari.services.add
        header: 'AMBARI_INFRA Service'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'AMBARI_INFRA'

      @ambari.services.wait
        header: 'AMBARI_INFRA Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'AMBARI_INFRA'

      @ambari.services.component_add
        header: 'INFRA_SOLR Add'
        if: options.post_component and options.takeover
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'INFRA_SOLR'
        service_name: 'AMBARI_INFRA'

      for host in options.instance_hosts
        @ambari.hosts.component_add
          header: 'INFRA_SOLR Host Add'
          if: options.post_component and options.takeover
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'INFRA_SOLR'
          hostname: host

## Dependencies

    ssh2fs = require 'ssh2-fs'
    {merge} = require 'nikita/lib/misc'