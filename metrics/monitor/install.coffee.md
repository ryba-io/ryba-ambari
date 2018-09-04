
# Ambari Metrics Monitor Install

    module.exports =  header: 'Ambari Metrics Monitor Install', handler: ({options}) ->
    
## Register

      @registry.register ['ambari','configs','default'], 'ryba-ambari-actions/lib/configs/set_default'
      @registry.register ['ambari','services','add'], 'ryba-ambari-actions/lib/services/add'
      @registry.register ['ambari','services','wait'], 'ryba-ambari-actions/lib/services/wait'
      @registry.register ['ambari','services','component_add'], 'ryba-ambari-actions/lib/services/component_add'
      @registry.register ['ambari', 'hosts', 'component_add'], "ryba-ambari-actions/lib/hosts/component_add"
      @registry.register ['ambari', 'hosts', 'component_install'], "ryba-ambari-actions/lib/hosts/component_install"
      @registry.register ['ambari', 'hosts', 'component_wait'], "ryba-ambari-actions/lib/hosts/component_wait"
      @registry.register ['ambari','kerberos','descriptor', 'update'], 'ryba-ambari-actions/lib/kerberos/descriptor/update'

## Packages

      @service
        name: 'ambari-metrics-monitor'

### METRICS_MONITOR component wait
Wait for the NODEMANAGER component to be declared on the host

      @ambari.hosts.component_wait
        header: 'METRICS_MONITOR WAITED'
        url: options.ambari_url
        # if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_MONITOR'
        hostname: options.fqdn

### METRICS_MONITOR component install
Put the METRICS_MONITOR component declared on the host as `INSTALLED` desired state

      @ambari.hosts.component_install
        header: 'METRICS_MONITOR set installed'
        url: options.ambari_url
        # if: options.takeover
        username: 'admin'
        password: options.ambari_admin_password
        cluster_name: options.cluster_name
        component_name: 'METRICS_MONITOR'
        hostname: options.fqdn
