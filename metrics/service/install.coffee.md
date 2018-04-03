
# Ambari Metrics Install

    module.exports =  header: 'Ambari Metrics Install', handler: (options) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Upload Default Configuration

      # @call -> console.log options.configurations
      @ambari.configs.default
        header: 'AMS Configuration'
        url: options.ambari_url
        if: options.post_component
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        stack_name: options.stack_name
        stack_version: options.stack_version
        discover: true
        configurations: options.configurations
        target_services: 'AMBARI_METRICS'

## Add AMBARI_METRICS Service

      @ambari.services.add
        header: 'AMBARI_METRICS Service'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'AMBARI_METRICS'

      @ambari.services.wait
        header: 'AMBARI_METRICS Service WAITED'
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        name: 'AMBARI_METRICS'

      @ambari.services.component_add
        header: 'METRICS_COLLECTOR Add'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_COLLECTOR'
        service_name: 'AMBARI_METRICS'

      @ambari.services.component_add
        header: 'METRICS_MONITOR Add'
        if: options.post_component
        url: options.ambari_url
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_MONITOR'
        service_name: 'AMBARI_METRICS'

## Install Component

      for host in options.monitor_hosts
        @ambari.hosts.component_add
          header: 'METRICS_MONITOR Host Add'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'METRICS_MONITOR'
          hostname: host

      for host in options.collector_hosts
        @ambari.hosts.component_add
          header: 'METRICS_COLLECTOR Host Add'
          if: options.post_component
          url: options.ambari_url
          username: 'admin'
          password: options.ambari_admin_password
          cluster_name: options.cluster_name
          component_name: 'METRICS_COLLECTOR'
          hostname: host

